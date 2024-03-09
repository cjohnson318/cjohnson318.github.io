---
layout: post
title:  "Mediator Pattern"
date:   2024-03-09 00:00:00 -0700
categories: python
---

The mediator pattern is a behavioral pattern from the Design Patterns book. The
main idea is that each component has a reference to some centralized mediator,
and can notify that mediator when some change occurs.

{% highlight python %}
class Mediator:

    def notify(self, sender: object, event: str):
        raise NotImplementedError()

class LoadMediator(Mediator):

    def notify(self, sender: object, event: str):
        if isinstance(sender, Load):
            print(f'Load is {event}')

class Load:

    def __init__(self, mediator):
        self.mediator = mediator

    def loaded(self):
        self.mediator.notify(self, 'loaded')

    def delivered(self):
        self.mediator.notify(self, 'delivered')
{% endhighlight %}

An example of application code using this pattern.

{% highlight python %}
mediator = LoadMediator()
load = Load(mediator)
load.loaded()
load.delivered()
{% endhighlight %}
