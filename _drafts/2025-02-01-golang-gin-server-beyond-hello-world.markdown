---
layout: post
title: "Taking a Go Gin Server Beyond Hello World"
date: 2025-02-01 00:00:00 -0700
tags: go
---

This represents a set of notes and observations about building a more modular
Gin RESTful API server in Go. I have a lot of experience building large backend
systems using Python and Django, and I wanted to apply some of the lessons I've
learned from Django to Gin.

## Accept and Return Simple Things

It's hard! I've used Go, I feel familiar with channels, and using structs and
interfaces instead of classes, but I haven't tried to take a Gin server and add
strict layers before. The first thing I learned, or re-remembered, was: try to take and return simple objects from functions, and keep the messy details
inside those functions.

for example, instead of accepting `(gin.Context).Request().Context` as a
function argument, just pass the simpler `gin.Context`, and unpack that the 
rest of what you need inside the function. Instead of returning a
`*mongo.Collection` object, keep unpacking that until you can return a
collection of the objects that you actually car about.

Said another way, focus on the interfaces in your logic, and keep the interfaces
as simple as possible. That way, it's easier to test your functions at those 
interfaces, and it's easier to refactor chunks of code.

Given expected input and output, there are many ways to create a set of
functions that transform the input into the output, but there are very few ways
that maximize the complexity within those functions, while minimizing the
complexity at the boundaries, while still remaining intuitive enough to explain
and remember, and that's the goal.

Really, it's the boundaries that define your solution. Once you can see the 
boundaries clearly, then the rest of the code writes itself.

## Balance Locality and Centrality

On a micro level, you want everything you need to look at to solve a problem to 
fit on a short page. On macro level, you want to be able to configure the whole
system from a short page.

{% highlight go %}

{% endhighlight %}

