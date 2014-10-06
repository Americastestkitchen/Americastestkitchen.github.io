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
rails new api -T -d postgresql
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

Set up a connection - for your backend and frontend to speak to eachother. for this we used the gem rack-cors [rack-cors githhub](https://github.com/cyu/rack-cors) "Rack::Cors provides support for Cross-Origin Resource Sharing (CORS) for Rack compatible web applications".

## Setting up the Ember-CLI Frontend
{% highlight ruby %}
config.assets.precompile += ['new_manifest.js', 'new_manifest.css']
{% endhighlight %}

## Setting up Chrome Extention 

