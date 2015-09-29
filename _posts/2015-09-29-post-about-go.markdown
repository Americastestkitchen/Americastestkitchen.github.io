---
layout: post
title:  "Go for a Beginner"
author: Janice K. N. Smith
github_username: jks8787
date:   2015-09-29 23:15:00
categories: go
---

First - definition of a beginner for the purpose of this post -- I started my time as a professional programmer almost 2 years ago. I was self taught mainly in python - followed by a term in a really great code bootcamp. I mainly develop in ruby on rails - with some backbone and react.

## What is Go?
The official answer is that Go, also called golang, is a programming language developed at Google in 2007 and is a statically typed language with garbage collection, type safety, some dynamic typing capabilities.

Also according to https://golang.org/:

> … it's worth trying again with a new language, a concurrent, garbage-collected language with fast compilation…
> <ul>
> <li>It is possible to compile a large Go program in a few seconds on a single computer.</li>
> <li>Go provides a model for software construction that makes dependency analysis easy and avoids much of the overhead of C-style include files and libraries.</li>
> <li>Go's type system has no hierarchy, so no time is spent defining the relationships between types. Also, although Go has static types the language attempts to make types feel lighter weight than in typical OO languages.</li>
> <li>Go is fully garbage-collected and provides fundamental support for concurrent execution and communication.</li>
> <li>By its design, Go proposes an approach for the construction of system software on multicore machines.</li>
></ul>

## Why learn Go?

* the syntax is easy to pick up - it feels simple and is easy to learn.
* really nice documentation help with the above point - https://golang.org/doc/faq is a nice place to start.
* It has garbage collection - this is great for someone like me who is used to this being handled for me -  In many compiled languages one has to manage on your own, such as in native C++ which by default has no such thing - you an always write your own or use a third party solution, but Go does it for me!
* go compiles very quickly - speed is king.
* I learned programming with dynamic typing. I had expected Go, being statically typed, to be more work painful to learn - especially after wrestling with tutorials for other languages . This was a great experience with static typing.
* Ruby's support for concurrency is something you need to set up with the help of a framework - like ["Celluloid"](http://celluloid.io/). In Go - Go channels provide a concurrency primitive comparable to the actors model, and channels come baked in already.

There are other great things about Go - but I am focusing on my short list.

## How to start?

* a tour of go - this is a really great place to start. ( https://tour.golang.org )
* a great place to look for tools - ( https://github.com/avelino/awesome-go )

For web development  - after trying a few different configurations of libraries - I found gin to be best for me. ( https://github.com/gin-gonic/gin ), it is well docuamented and cuts down on a lot of boiler plate code. That said a great thing about the Go ecosystem is there are several different options.
