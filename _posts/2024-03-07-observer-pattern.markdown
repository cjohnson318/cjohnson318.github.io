---
layout: post
title:  "Observer Pattern"
date:   2024-03-07 00:00:00 -0700
categories: python
---

This is an example of the Observer Pattern. This can be used to simplify
complex logic by using a pub/sub system to notify objects about changes to the
state of some central object.

In this example, I'm thinking of trucks coming into a warehouse. When a truck
comes in, forklifts need to be sent to the correct bay in order to unload the
truck, and invoices for the shipment need to be paid.

{% highlight python %}
class EventManager:

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
event_manager = EventManager()
warehouse = Warehouse(event_manager)

forklift_pool = ForkliftPoolListener(num_forklifts=3)
invoicing = InvoiceListener()

event_manager.subscribe("truck_arrived", forklift_pool)
event_manager.subscribe("truck_arrived", invoicing)

warehouse.truck_arrived(bay=23)

event_manager.unsubscribe("truck_arrived", forklift_pool)

warehouse.truck_arrived(bay=17)
{% endhighlight %}

