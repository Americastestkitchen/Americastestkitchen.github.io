---
layout: post
title:  'The Value of Feedback: A Tale of Building An Internal Tool'
author: Patrick Hereford
github_username: phereford
date:   2015-07-15 23:15:00
categories: process tooling
---

_As most of us software engineers have known, tooling is a very important part of
our day. In fact, there was an
[eloquent post](https://medium.com/@leeb/why-invest-in-tools-3240ce289930) on
this subject matter recently. We are not the only discipline that benefit from
such tools. Below is a tale wrought with insight on how an iterative feedback
cycle influenced a tool we created for our WebEdit team to increase their
productivity while creating unstructured content._

## Problem Scoping
*Background*: At America's Test Kitchen, we are working on a new content type
(internally called an `Article`) that is more complicated than our current
content offerings. Instead of having highly structured data, we are designing
this to be free flowing (think *one wysiwyg to rule them all*).
  
*Stack*: Our stack for this piece of content is a Ruby on Rails backend with a
shiny new React frontend utilizing both [flux](https://facebook.github.io/flux/)
and [react-router](https://github.com/rackt/react-router).  
  
*Problem*: Our WebEdit team, the people that create all of our wonderful web 
content, need sophisticated previewing tools for this unstructured offering.
And that is what we are building, a semi-realtime previewing engine that gives
content creators _instantaneous_ feedback for their free flowing content.  
  

## Minimum Viable Product
Our prototype is going to be a proof of concept. Technologies used:
`JSXTransformer`, `iframe`, `jQuery`, `React`. In our current CMS (created with
bootstrap), we are going to have a preview button that brings said `iframe` into
view with the click of a button. As someone changes the one html field, it will
re-render the iframe and reload the entire page with the new data (similar
to CodePen.io).
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/1OhoJW5GqFM" frameborder="0" allowfullscreen></iframe>  
  
Holy smokes Batman! Well, we are off to a great start. You can see that as we 
type, we have a request firing off to reload the page with the new value. The
`ArticlePreviewContainer` then transpiles the JSX and evaluates it.  
  
The original thinking behind this was to make it similar to CodePen using some
[Base64 shenanigans gist](https://gist.github.com/ncerminara/11257943) and created
both a Rails route and React route both looking like: `/admin/article/:id/:base`.
Using the gist above, we are encoding the value of the body input into `Base64`
and passing it to the routes for the `:base` parameter.  We even having it 
working with one of our custom React components! Hot Sauce! Here is what our 
React Preview Component looks like:

{% gist 92afacec15a159292256 %}  
  
You might ask yourself, "What's up with that RecipeIndexPage require at the top?"
Currently, it is the only custom component I am allowing to be rendered in the
`ArticlePreviewContainer` React component. Requiring it was the only way I
thought to show WebEdit what it looks like when using custom components.  
  
Speaking of WebEdit, it's time to meet with them to get their thoughts on this.

## MVP - Feedback Cycle
_(Just going to put my scribbled down notes from WebEdit below...)_  
- This is really cool, but does it have to be inside the same window? Can we 
put it in it's own window?  
- Can we get some sort of error message if we write bad markup (jsx)?  
- Can we get documentation for the various components we have access too?  
  
_(Some notes I took down when dogfooding it to the engineering team...)_  
- Iframe? Wat? Why aren't you using postMessage?  
- We should definitely whitelist things that they have access too using 
`React.Children.map` for this content.  
- `JSXTransformer` is being deprecated. Use Babel and pass over the transpiledJS  
  
Woah. All great, great ideas. Definitely going to incorporate this feedback into
my MVP.
  
## Version 2 - The previewWindow and postMessage...and linting?
Now here is where the fun begins!  
  
Technologies used:
[Babel browser.js](https://github.com/babel/babel/blob/master/src/babel/api/browser.js),
[eslint](https://github.com/eslint/eslint),
[Underscore](http://underscorejs.org/),
[postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage).  
  
Technologies being removed: `iframe`, `JSXTransformer`, `Base64 gist`  
  
The only real code to care about here is:  
{% gist fe02ffe6f8a39a3668ef %}  
  
We register a keyup event on the singular input field with a delay using 
Underscore's debounce method. When the callback is invoked, it performs the
following functions:  
1) Lints the JSX.  
2a) On failure, disable all submit buttons and display an error message.  
2b On success, enable all submit buttons and transform the jsx using 
babel (and wrap it in an `Article` react component for...reasons).  
3) After transformation, it checks for our previewWindow and posts the message
to the previewWindow.  

Here is what it looks like:  
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/YO7VZx0zjw0" frameborder="0" allowfullscreen></iframe>  
   
Time for one more round of feedback!  
  
## Version 2 - Feedback Cycle  
_(Going back to my scribbled down notes from WebEdit...)_  
- This is much better. But when I update the title, it doesn't update!  
- We still need a style guide to let us know what the markup looks like and
what it outputs.  
  
_(Going back to my scribbled down notes from engineering...)_  
- Where is my white list!!?!?  
- Where is the friggin white list??  
- Moar whitelist please!  
  
Alright...thanks guys. I can feel the feature already maturing as I received no
feedback on the actual implementation of it and mostly received feedback on 
evolving it and protecting WebEdit where I can (whitelisting). We are definitely
on the right track, so let's start moving onto version 3!  
  
## Version 3 - Final Final Release (Maybe?)
So instead of listening to a single input (on 'keyup'), we need to bind our 
postMessage event to more inputs.  
  
Here is the code for said event passing json and listening to _specific_ inputs.
{% gist d35e6fcda00b68301c74 %}  
  
Whitelisting didn't prove very difficult either. We created a mixin that would
traverse the component's children and return null if it was 
not in the list (hence why we wrapped the resultant JSX in `<Article>` tags).
Currently, it only supports whitelisting of HTML elements that React supports.
We will be modifying the conditional logic to also check for a whitelist of 
custom components in the very near future.  
  
### Whitelist:  
{% gist e501bdd75b8971f84975 %}  
  
### Article Component Whitelisting:  
{% gist b8e91ccdb26d7e6b7337 %}  
  
What's this look like now?  
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/3mkwPOC_uYo" frameborder="0" allowfullscreen></iframe>  
  
As for the documentation, we found an
[open sourced project](https://github.com/jmfurlott/react-styleguide) that has
been amazing for conveying both markup and the actual look of the component.  
  

## Conclusion
Without a doubt the feedback cycle from both the end user of this tool (WebEdit)
and from other engineers vastly improved the quality and maintainability of the
tool. Going from MVP to something much more robust and maintainable can only be
realized by getting feedback early and often as features are worked on.  
  
Most importantly, WebEdit can now be highly productive in using this near
real-time tool to build really amazing content.
