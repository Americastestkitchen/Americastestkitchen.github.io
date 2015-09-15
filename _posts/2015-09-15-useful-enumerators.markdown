---
layout: post
title:  "You Gotta Check Out This Enumerator, It'll Change Your Life I Swear!"
author: Chris Candelora
github_username: ccandelora
date:   2015-09-15 23:15:00
categories: enumerators
---

Well, not really but it has made my job easier and I think it could do the same for you.

I'm relatively new to Ruby and in my past life my bread and butter language was PHP. When I was told that our stacks were going to be ripped down and replaced with Ruby on Rails I had mixed emotions. In one hand I was really excited to be working with Ruby on Rails. A new cool language to sink my teeth into. On the other hand I was going to be working with Ruby on Rails! I thought, OK maybe this wont be so bad. I can use what I know about PHP and apply it to Ruby on Rails.  So I hit the books and tutorials and anything I could get my hands on. I quickly learned that Ruby on Rails and Ruby had a lot of built in magic and PHP had none of that. It took me a long time to get my head around it. A colleague told me to just "believe in the magic". Well, now I believe and I would like to share some of that "magic" with you.

One of the events that really help me believe was a suggestion from another colleague. He told me about this blog called ["Ruby Tapas"](http://www.rubytapas.com/) by [Avdi Grimm](http://devblog.avdi.org/). You have to check it out! I can't say enough good things about it but maybe I'll save that for another blog post. Everyday I would go through his posts while commuting to work. I would sit on the train with my headphones on staring at my phone and continually saying out loud "ah that's cool!". My fellow commuters started to sit a little further away from me each day.

One of those "ah that's cool!" moments came after I saw Avdi's post about this strange enumerator called partition.

## What is partition and what does it do?

From [apidock](http://apidock.com/ruby/Enumerable/partition)

"partition() public

Returns two arrays, the first containing the elements of enum for which the block evaluates to true, the second containing the rest."

In simpler terms it splits a collection into two subsets based on some criteria.

A few days later after stumbling on this gem I was tasked with building a tool for our Web Edit Team. This team will sometimes free up content on our sites and they needed a quick and easy way to find out which pieces of content are currently free. The bit of logic we have built into our code to determine if a piece of content is free doesn't use a status attribute but instead uses start and end dates to determine if something is currently free. I thought this would be so much easier if we just had a status attribute and I could easily just query everything that had a status of active. I thought, I could make a schema change and add the new attribute and run a daily job to check and alter the status but that just seemed like crazy town. Then I thought about the Ruby "magic" and the post I saw a few days earlier about partition. Everything just fell into place. I didn't need to make a crazy schema change it was just for a view after all. Partition was exactly what I needed.

So I built a helper that would take an array of hashes and easily split the array into to two arrays. An active array and an inactive array.

{% gist b4c06cc1e60351658bc9 %}

The next part was even easier. I just needed to create a view that called the helper and returned the instance variables @active and @inactive. Once I checked for the presence of the instance variables all that was left to do was iterate over the arrays.

{% gist 9c5cba7691834b525bef %}

So the next time you find yourself needing to split a collection into two subsets based on some criteria just remember this story and the "magic" of partition.