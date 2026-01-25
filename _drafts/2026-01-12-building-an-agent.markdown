---
layout: post
title: "Building an Agent"
date: 2026-01-12 00:00:00 -0700
tags: python ai
---

Some notes on building an AI agent.


## The OpenAI API

OpenAI created a popular format for talking to LLMs. It looks like this:

{% highlight python %}
import requests

response = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": "Bearer YOUR_API_KEY"},
    json={
        "model": "gpt-4",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Hello!"}
        ]
    }
)

print(response.json()["choices"][0]["message"]["content"])
# Output: "Hello! How can I help you today?"
{% endhighlight %}

Why this matters: This format became a standard. Many tools now speak "OpenAI
API format" even if they're not OpenAI. This means you can swap out the 
underlying LLM without changing your code.


## vLLM

{% highlight python %}
{% endhighlight %}

