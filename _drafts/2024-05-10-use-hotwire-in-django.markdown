---
layout: post
title:  "Use Hotwire in Django"
date:   2024-05-10 00:00:00 -0700
tags: python
---
<html>
Hotwire is a collection of technologies developed to give server-side rendered sites the speed and interactivity of single-page applications.

There are several pieces of Hotwire: Turbo Drive, Turbo Frames, Turbo Streams.

> TLDR: Turbo Drive focuses on `<a>` and `<form>` elements.

Turbo Drive intercepts clicks on `<a>` tags and `<form>` submissions and transforms those from full HTTP requests to AJAX requests. When the request is fulfilled, the new `<head>` is merged with the previous, and the entire `<body>` is swapped out. This means that assets that were fetched when you loaded the page aren't reloaded.

As an example of all of this, I'm going to create a very simple Django application with a single app called "chart". This app is going to server pages with charts, rendered by [Chart.js](https://www.chartjs.org/). I'm interested in this because I want to use Hotwire technology to make Chart.js as reactive as [plotly](https://plotly.com/).

<div class="nojekyll">
<code>
{% load static tailwind_tags %}
</code>
</div>
</html>