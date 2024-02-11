---
layout: post
title:  "Simple Ollama Example"
date:   2024-02-10 19:00:00 -0700
categories: llm
---

Ollama is an incredibly easy tool to use. To get a local model that you can customize with a prompt run:

{% highlight console %}
ollama pull <model>
{% endhighlight %}

{% highlight console %}
ollama -h
ollama rm wellname
vim wellname-model-file
ollama create wellname -f wellname-model-file
ollama run wellname
{% endhighlight %}

