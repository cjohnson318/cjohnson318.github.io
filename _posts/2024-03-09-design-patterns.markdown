---
layout: post
title: 'Design Patterns'
date: 2024-03-09 00:00:00 -0700
categories: tech
tags: python
---

Here I'll discuss some design patterns I've used in previous projects.


## Mediator Pattern

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


## Observer Pattern

The observer pattern is a behavioral pattern from the Design Patterns book. The
main idea is that each component has a refence to some centralized observer,
and can notify the observer when some change occurs. The difference from the
mediator pattern is that the reaction to the observed event, handled by
different listener classes managed by the observer, can be turned on and off at
runtime through a subscribe/unsubscribe feature.

In this example, I'm thinking of trucks coming into a warehouse. When a truck
comes in, forklifts need to be sent to the correct bay in order to unload the
truck, and invoices for the shipment need to be paid.

{% highlight python %}
class Observer:

    def subscribe(self, event: str, listener):
        raise NotImplementedError()

    def unsubscribe(self, event: str, listener):
        raise NotImplementedError()

    def notify(self, event: str, data):
        raise NotImplementedError()

class WarehouseEventManager(Observer):

    def __init__(self):
        self.listeners = dict()

    def subscribe(self, event: str, listener):
        if event in self.listeners:
            self.listeners[event].append(listener)
        else:
            self.listeners[event] = [listener]

    def unsubscribe(self, event: str, listener):
        if event in self.listeners:
            if listener in self.listeners[event]:
                i = self.listeners[event].index(listener)
                self.listeners[event].pop(i)

    def notify(self, event: str, data):
        if not event in self.listeners:
            return
        for listener in self.listeners.get(event):
            listener.update(data)

class Warehouse:

    def __init__(self, event_manager: EventManager):
        self.event_manager = event_manager

    def truck_arrived(self, bay):
        # update
        self.event_manager.notify("truck_arrived", bay)

class EventListenerInterface:

    def update(self, data):
        raise NotImplementedError()

class ForkliftPoolListener(EventListenerInterface):

    def __init__(self, num_forklifts: int):
        self.num_forklifts = num_forklifts

    def update(self, data):
        print(f'Sending {self.num_forklifts} forklift operator(s) to bay {data}')

class InvoiceListener(EventListenerInterface):

    def update(self, data):
        print(f'Paying invoice for shipment id on bay {data}')

{% endhighlight %}

These objects can then be used in application code as:

{% highlight python %}
event_manager = WarehouseEventManager()
warehouse = Warehouse(event_manager)

forklift_pool = ForkliftPoolListener(num_forklifts=3)
invoicing = InvoiceListener()

event_manager.subscribe("truck_arrived", forklift_pool)
event_manager.subscribe("truck_arrived", invoicing)

warehouse.truck_arrived(bay=23)

event_manager.unsubscribe("truck_arrived", forklift_pool)

warehouse.truck_arrived(bay=17)
{% endhighlight %}

And this should be the output of that code:

{% highlight python %}
Sending 3 forklift operator(s) to bay 23
Paying invoice for shipment id on bay 23
Paying invoice for shipment id on bay 17
{% endhighlight %}
