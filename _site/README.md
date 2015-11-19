# ATK Developer Blog

This is the America's Test Kitchen Developer Blog. We'll post about things we're doing in tech here. From OSS to silly hardware hacks for our office.

## Setting up Locally

```sh
gem install jekyll
rbenv rehash  # If you're using rbenv (you should be)
git clone https://github.com/Americastestkitchen/Americastestkitchen.github.io
```

## Writing a Post

```sh
git checkout -b post-about-nothing
touch _posts/2014-04-12-post-about-nothing.markdown
```

Edit the post, don't forget the front matter. Keep in mind the categories go into the URL of the post, so keep them consistent and short.
```
---
layout: post
title: "A Post about Nothing"
author: Nathan Lilienthal
date: 2014-04-12 23:15:00
categories: nothing
---
```

You can optionally add the following attributes to the page.

```
github: github: Americastestkitchen/hostel
github_username: redneckbeard
meta: Some words...
```

Run the site locally.

```sh
jekyll serve -w  # Automatically updates as you write.
```