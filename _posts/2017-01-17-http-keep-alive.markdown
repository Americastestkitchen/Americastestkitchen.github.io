---
layout: post
title: "Troubleshooting Node Http.request timeout errors in an isomorphic React application"
author: Jason Aslakson
date: 2017-01-17 11:15:00
categories: node
---

## Background

In June of 2016 ATK relaunched the americastestkitchen.com website as an Node/React isomorphic single page application. The project was extremely challenging and the timeline was tight but we were able to deliver a solid product that met the technical requirements set forth by the business.

Overall, the deployment went extremely smoothly with very little in the way of unexpected developments. As developers we enjoyed working with React and Node. We were able to build a ton of functionality in the tight timeline afforded by the business. In September, as our seasonal traffic started picking up, our app began sporadically responding to clients with 500 errors. This is where the story begins...

## The Problem

Looking through the Heroku logs, we saw the following errors:

```
Unexpected error in API { FetchError: request to
  https://www.americastestkitchen.com/some-api-endpoint failed,
  reason: connect ETIMEDOUT
```

Looking through some SO posts, we found a post that seemed promising. It had the following comment:

```
By default, Node has 4 workers to resolve DNS queries.
If your DNS query takes long-ish time, requests will block on the
DNS phase, and the symptom is exactly ESOCKETTIMEDOUT or ETIMEDOUT.

Try increasing your uv thread pool size:

export UV_THREADPOOL_SIZE=128
```

This sounded reasonable and local testing indicated that changing this setting didn't have an adverse effect on app performance. Once deployed to production, however, we quickly realized that this was a false start. The change did not fix the problem. Back to the investigation...

At this point we were also seeing an H13 error in our Heroku logs. From Heroku's error documentation "This error is thrown when a process in your web dyno accepts a connection, but then closes the socket without writing anything to it."

Example H13 error
```
2010-10-06T21:51:37-07:00 heroku[router]: at=error code=H13
desc="Connection closed without response" method=GET path="/"
host=myapp.herokuapp.com fwd=17.17.17.17 dyno=web.1 connect=3030ms
service=9767ms status=503 bytes=0
```

After trying a quick-fix we realized that while we searched for a fix we had to, at the very least, provide a slightly better error page for our users. The 500 errors that they were seeing were being displayed after the max-timeout was surpassed - which was 30 seconds! Additionally, the error page was the standard Heroku error page which does not inspire confidence. So, we set out to implement a custom timeout response and install some extra logging to help us gather some additional data.

To do this, we needed to inject some custom middleware in our Express pipeline. To start, we set the timeout at 25s, 5s short of the Heroku timeout.

In our main express application file for our front-end code:
```
app.use(timeout('25000'));
```

The timeout code (TimeOutPage is a React component that renders our error page):
```
function onTimeout(delay, cb, req, res) {
  return () => {
    const {
      cookies: { ... },
      headers: { 'user-agent': user_agent },
      url
    } = req;
    const agent = useragent.parse(user_agent);
    console.error({
      url,
      user_agent: agent.toString()
    });
    const html = ReactDOMServer.renderToString(
      <TimeOutPage />
    );
    cb(res.status(503).send(html));
  };
}

/**
 * Copied and updated from https://github.com/expressjs/timeout
 * to allow us to render an error template instead of a plain string
 * error and to log data to console.
 */
export default (time, options) => {
  const opts = options || {};
  const delay = Number(time || 5000);
  const respond = opts.respond === undefined || opts.respond === true;

  return (req, res, next) => {
    const id = setTimeout(() => {
      req.timedout = true;
      req.emit('timeout', delay);
    }, delay);
    if (respond) {
      req.on('timeout', onTimeout(delay, next, req, res));
    }
    req.clearTimeout = () => clearTimeout(id);
    req.timedout = false;
    onFinished(res, () => clearTimeout(id));
    onHeaders(res, () => clearTimeout(id));
    next();
  };
};
```

## The Solution

Now that we had installed a bit of a backstop for that ugly 500 error page, we focused on the H13 error. Our Node application connects to our api for data retrieval. On each page request there could be several http connections from Node to the api. The description of the H13 lead us to believe that our application was connecting to the API and then getting all gummed up. Sounds like we need to do a better job of managing these connections.

Looking into the Node documentation for the http/https module (where we should have started in the first place - RTFM right?) we find some interesting options for managing our connections. In particular, the [http.Agent](https://nodejs.org/api/http.html#http_class_http_agent) class was standing out. By default, http requests from Node can, but are not required to, have a custom `Agent` property on the `options` argument for an `http.request`. By omitting your own `Agent` option we were accepting the defaults. The answer was staring us in the face.

From the `Agent` documentation:
```
const http = require('http');
var keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
```

The key here is the `{ keepAlive: true }`. As described by the documentation: *"Keep sockets around in a pool to be used by other requests in the future"*.

Deploying this change was the solution to our problem. I was still bothered by the fact that our Node app wouldn't retry a request if it received an initial failure and that it would try for (by default with Heroku) 30 seconds before the connection was killed. There were certainly times where a connection would go astray on the first try and work perfectly well on the second. We don't want to wait around for 30 seconds to find out that we should have just tried a second time. To address this concern, we added a `timeout` option to our `http.request` options and then in our response processing, we looked for timeouts and issued a retry.

Our node application uses the `isomorphic-fetch` package, which is an abstraction layer on top of the `http.request` module. So, here is an approximation of our code (shortened and simplified):

```
let response;
let statusCode;
const fetchOptions = {
  agent: new http.Agent({ keepAlive: true }),
  timeout: 5000,
};

...setup code for our request...

response = await fetch(url, fetchOptions).catch(catchError);
statusCode = response.status;

...other code looking for errors...

// retry once for timeouts
if (statusCode === 503) {
  console.error('RETRY', url);
  response = await fetch(url, fetchOptions).catch(catchError);
  statusCode = response.status;
}
```

With this new code, we were now able to pool our sockets which fixed the timeout issues and we were able to retry our requests in the case of a temporary blip. In the end we wish that we had addressed these issues prior to launch but the troubleshooting process was a great learning experience. In my years as a software engineer I have learned that no matter how diligent you are in your process and review you can't catch everything. The key is to learn from your mistakes and improve for next time.
