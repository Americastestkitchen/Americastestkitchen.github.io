---
layout: post
title:  "All about that Buildpack"
author: Nicole Pupillo
github_username: npupatk
date:   2015-12-31 23:59:59
categories: buildpack
---

While upgrading our legacy production Wordpress blog app "The Feed" from Heroku stack Cedar-10 to Cedar-14, I had the opportunity to work with Heroku buildpacks.

## What's a buildpack?
Buildpacks are a tool for modifying how a Heroku app is built & deployed. For Heroku to support many languages & tools it tries to be an agnostic platform with no native support for any one language. Buildpacks can be managed as an independent GitHub repo. They behave like an adapter for customizing an app's build process - in so far that a language can even be completely contained inside a buildpack if needed. In my scenario I used buildpacks to add operating system level dependencies that were missing from a new Ubuntu OS of the underlying Heroku app.

## What's in a buildpack?
At a minimum in the root folder of the buildpack repo, a buildpack contains a bin directory with three Bash scripts: detect, compile, release:

* *bin/detect* - used to check if a specific file exists in order to continue or else will exit.
--Example, from Apache+PHP buildpack for Wordpress [https://github.com/Americastestkitchen/heroku-wordpress-php](https://github.com/Americastestkitchen/heroku-wordpress-php): here we check for the existence of an index.php file & output a happy or sad message visible during the Heroku build, for example:

    `#!/usr/bin/env bash`<br>
    `if [ -f $1/index.php ]; then`<br>
      `echo "index.php ATK heroku-wordpress-php" && exit 0`<br>
    `else`<br>
      `echo "index.php not detected" && exit 1`<br>
    `fi`<br>

* *bin/compile* - the bulk of the work that modifies the heroku slug (more about slugs here: [https://devcenter.heroku.com/articles/slug-compiler](https://devcenter.heroku.com/articles/slug-compiler). For example: compile specifies where binaries are installed, Heroku config files are written, dependencies are resolved & installed, & static files are built.
* *bin/release* - modifies the heroku release (more about releases here: [https://devcenter.heroku.com/articles/releases](https://devcenter.heroku.com/articles/releases))


## The Challenge
I created these additional buildpacks to assist the existing heroku-wordpress-php buildpack & the new heroku-buildpack-apt buildpack because Heroku's Cedar-14 stack was missing some OS level dependencies, specifically libjpeg62 & libssl0.9.8. Heroku confirmed my issue in their documentation that certain Ubuntu packages were now missing from the new Cedar-14 stack (see [Ubuntu Packages on Cedar and Cedar-14](https://devcenter.heroku.com/articles/cedar-ubuntu-packages)).

In my scenario, the existing state of the app included two GitHub repos:

* The Feed - our private Wordpress blog repo
* Apache+PHP buildpack for Wordpress [https://github.com/Americastestkitchen/heroku-wordpress-php](https://github.com/Americastestkitchen/heroku-wordpress-php)

And by the end, the app included these additional repos:

* ATK heroku-buildpack-apt [https://github.com/Americastestkitchen/heroku-buildpack-apt](https://github.com/Americastestkitchen/heroku-buildpack-apt) -forked from [ddollar/heroku-buildpack-apt](https://github.com/ddollar/heroku-buildpack-apt)
* ATK heroku-buildpack-multi [https://github.com/Americastestkitchen/heroku-buildpack-multi](https://github.com/Americastestkitchen/heroku-buildpack-multi) -forked from [heroku/heroku-buildpack-multi](https://github.com/heroku/heroku-buildpack-multi)

## The Solution
For my solution I set *heroku-buildpack-multi* as the primary buildpack of The Feed repo, then *heroku-buildpack-multi* loads the two other buildpacks in the order specified in the .buildpacks file of The Feed repo.<br>
.buildpacks file: <br>
`heroku-buildpack-apt`<br>
`heroku-wordpress-php`

* *heroku-buildpack-apt* - This buildpack installs the missing Ubuntu system dependencies when Heroku is building the app. These dependencies are specified on separate lines in the *Aptfile* file of The Feed repo. <br>
Aptfile: <br>
`libjpeg62`<br>
`libssl0.9.8`

* *heroku-wordpress-php* - This is the existing buildpack which installs a specific version of Apache & PHP for our specific version of Wordpress.

## Keep in mind
--Since making changes to a buildpack affects the Heroku build process itself, testing any changes require re-deploying the app to Heroku. Use a non production test app for developing and testing your theories.

--You can set an app's primary buildpack in the CLI vs Heroku UI. Setting the buildpack in the Heroku app's Settings in the [Heroku Dashboard](http://dashboard.heroku.com) (under Reveal Config Vars) will set the buildpack as the default. However if a buildpack is set in the CLI it will take precedence over the one in the Heroku Dashboard but unfortunately will NOT visually change it in the Heroku Dashboard. Therefore I set the primary buildpack to heroku-buildpack-multi in both the Heroku Settings UI & the CLI to ensure consistency and avoid any confusion.

## Helpful CLI commands

--Clear the buildpacks<br>
`heroku buildpacks:clear -a your-app-name`

--Set a buildpack<br>
`heroku buildpacks:set https://github.com/heroku/heroku-buildpack-multi.git -a your-app-name`

--Display a full list of options for the buildpacks command<br>
`heroku help buildpacks`

--Double check which buildpacks are set via the CLI with the heroku buildpacks command<br>
`heroku buildpacks -a your-app-name`

--List an app's Heroku release history - useful if you need to roll back to a previous release<br>
'heroku releases -a your-app-name'

## Other uses
Another possible use of buildpacks I stumbled upon was to include command utilities for troubleshooting which are not included automatically in the Heroku build. So for example if you wanted to troubleshoot an app's network activity, you could have a buildpack install *traceroute* during the build process so it can be used in the app's Heroku bash console.
