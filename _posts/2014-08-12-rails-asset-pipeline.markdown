---
layout: post
title:  "Rails Asset Pipeline"
github: Americastestkitchen/presentations
author: Patrick Hereford
github_username: phereford
date:   2014-08-14 23:15:00
categories: rails asset_pipeline
---

## Slight Asset Pipeline Overview
The Rails Asset Pipeline can be slightly tricky to newcomers and those not in
the rails world. It was brought into this world when version 3.1 launched and
its main mission was to ease the developer pain on getting assets (images,
scripts, stylesheets) production ready. 

There are a lot of minor gotchas with the pipeline like:

- Remembering to actually use the helper methods to get things properly fingerprinted
- Remembering to add custom named manifest files to config/application.rb

```ruby
#config/application.rb
config.assets.precompile += ['new_manifest.js', 'new_manifest.css']
```

- Remembering to include newly created javascripts and stylesheets to our manifests.
- And I am sure there is much more that I am forgetting.

## Our Code and The Pipeline
Our codebase is a multi-tenant platform that services our top three sites:
[Cook's Illustrated](http://www.cooksillustrated.com),
[Cook's Country](http://www.cookscountry.com), and
[America's Test Kitchen](http://www.americastestkitchen.com). We have approximately
six-ish custom manifest files due to the multi-tenant nature of our platform. At
times, we do tend to forget to use one of the many helpers that Rails comes with.

## Quick Presentation For Knowledge Flow
In lieu of this lack of remembering to use helpers, I decided to create a quick
presentation (5-10minute talk in length) to go over the asset pipeline, less in
depth than 
[Rails documentation](http://guides.rubyonrails.org/asset_pipeline.html). I used
[cleaver](https://github.com/jdan/cleaver) to generate the presentation. Comments
and questions are welcomed.
