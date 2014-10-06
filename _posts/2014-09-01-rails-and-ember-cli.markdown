---
layout: post
title:  "Ruby on Rails as an API - for an Ember-CLI Frontend"
github: Americastestkitchen/presentations
author: Janice K. N. Smith 
github_username: jks8787
date:   2014-09-01 23:15:00
categories: rails ember-cli chrome_extension
---

## About the Application
this app was consived and put together for a reacent hack-a-thon - the Travel & Education Hackathon with LearnLaunch in Boston ( which was a 24 hour hackathon!). I was part of a team of 3 developers and one business person. The team was me Janice Smith, Alvin Crespo, and Ilan Rimon as devs and - Eli Meyer as the business end. I really enjoyed working with all of three of them and I very happy with the quality of the event over all. The point of the application was to have a low friction way to organize reaseach on a topics for yourself or to share with others. 

This post is really an overview of the technical bones of the project - uitlizing it the app ( we called it reseachR ) and an example application.

## Setting up a Ruby on Rails Backend
this will be out API (application programming interface). - Try to stay away from using too many generators - you will not be needed any views or need to respond to html. All those duites will be carried by the front end application. 

{% highlight shell %}
rails new backend -T -d postgresql
{% endhighlight %}

The -T to not use TestUnit and the -d postgresql because you to use a PostgreSQL database.

You need to set up models and controllers - for this App we needed models for Entries, Projects and Users. Be sure to setup controllers to serve up json.

You will want some serializers - to make sure your json is as will be execpted by the fontend App.

{% highlight ruby %}
class EntrySerializer < ActiveModel::Serializer
	attributes :id, :url, :note, :priority, :project
end
{% endhighlight %}

You will need some authenication for users - we used the Devise gem - set-up Devise as you would normally (their docs are pretty straightforward). Dont worry about exporting the views - you can create some seed data and check that auth is working now if you like.

{% highlight ruby %}
  code
{% endhighlight %}

Set up a connection - for your backend and frontend to speak to eachother. for this we used the gem rack-cors [rack-cors github](https://github.com/cyu/rack-cors) "Rack::Cors provides support for Cross-Origin Resource Sharing (CORS) for Rack compatible web applications".

## Setting up the Ember-CLI Frontend
Now you get to setup a new shiney ember-cli app!

if you have brew doctor all setup then:

{% highlight shell %}
brew install node
{% endhighlight %}

once this is done you will have node.js and you should have npm both installed.

{% highlight shell %}
npm insall -g bower
nmp insatll -g ember-cli
ember new frontend
{% endhighlight %}

this is where I tell you that - you should use the latest versions of all of these - and then take a look at the docs - ember-cli can be a lot of fun but try to be sure you are setting up your app in a way that does not fight the framework (this will make your life simpler - and if you find you have to do it - maybe pick a different tool set or frame work - there are a lot of good ones out there). 

You will need to set up your routes, each one represents a state in your app.
In our  router.js file - we had:
{% highlight hbs %}
import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
  this.route('application');
  this.route('protected');
  this.route('login');
  this.route('signup');

  this.route('entries');
  this.resource('projects');
});

export default Router;
{% endhighlight %}

Then you will set up your contollers in your new app - controllers  - controllers let you decorate models with display logic.

for our App  this was pretty boiler plate.

{% highlight hbs %}
import Ember from 'ember';
export default Ember.ArrayController.extend({
  exists: function() {
    if(this.get('content.length') > 0){
      return true;
    }
    return false;
  }.property('content.length')
});
{% endhighlight %}

Now you should be thinking about what you really want out of your app. You can set up some views are key because they are:  “The role of the view in an Ember.js application is to translate browser events into events that have meaning” [ember.js docs](http://emberjs.com/guides/views/)

We didn't use any views - because 24 hours - but these can be really powerful.

About models - "In Ember, every route has an associated model. This model is set by implementing a route's model hook, by passing the model as an argument to {{link-to}}, or by calling a route's transitionTo() method." [ember.js docs](http://emberjs.com/guides/models/)

We didn't need to do anything special to for our App - later of course we might need to make more changes. 

Now you need templates! “Handlebars templates as an HTML-like ... for describing the user interface”[ember.js docs](http://emberjs.com/guides/templates/)

our longin.hbs template:
{% highlight hbs %}
<form {{action 'authenticate' on='submit'}} role="form">
  <div class="form-group">
    <label for="identification">Login</label>
    {{input id='identification' placeholder='Enter Login' value=identification classNames="form-control"}}
  </div>
  <div class="form-group">
    <label for="password">Password</label>
    {{input id='password' placeholder='Enter Password' type='password' value=password classNames="form-control"}}
  </div>
  <button type="submit" class="btn btn-primary">Login</button>
</form>
{% endhighlight %}

Now you need to move on to authenication! 
We used the packages ember-cli-simple-auth to for the frontend App combined with  -auth-devise to hook up to devise in rails-land.

[ember-cli-simple-auth](https://github.com/simplabs/ember-cli-simple-auth)
[ember-cli-simple-auth-devise](https://github.com/simplabs/ember-cli-simple-auth-devise)

{% highlight shell %}
npm install --save-dev ember-cli-simple-auth
npm install --save-dev ember-cli-simple-auth-devise
{% endhighlight %}

## Setting up Chrome Extention 

Now you need to wrap it all togther - in our App reseachR - we wanted to be sure it would simple for the user - what better way to do that than a chrome extension. Lucky for us this is pretty simple. 

1- create a manifest file named manifest.json
2- add resources (must exist inside the extension package)
3- load unpacked extension
4- reload 

(see the [docs](https://developer.chrome.com/extensions/getstarted))

Lastly - if you want to see the code take a look on github [researchR](https://github.com/researchr).

   /)_/)
  ( .  .)
 C(") (")

