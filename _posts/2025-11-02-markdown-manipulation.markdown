---
layout: post
title: "Markdown Manipulation"
date: 2025-11-02 00:00:00 -0700
tags: linux
---

These are some running notes on using markdown and associated tools.

You can convert from markdown to HTML using pandoc or mulitmarkdown

{% highlight console %}
pandoc main.md > main.html
multimarkdown main.md > main.html
{% endhighlight %}

You if all of your, possibly interconnected, markdown docs have been converted
to HTML, then you can render them and follow the links using the `lynx`
utility.

{% highlight console %}
lynx main.html
{% endhighlight %}

You can also render a markdown directly using

{% highlight console %}
multimarkdown main.md | lynx -stdin
{% endhighlight %}
