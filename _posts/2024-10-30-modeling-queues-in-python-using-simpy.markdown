---
layout: post
title: "Modeling Queues in Python Using SimPy"
date: 2024-10-30 00:00:00 -0700
categories: tech
tags: python
---

I first read about modeling queues in 2014, but I did not appreciate the power
this abstraction until I started thinking about logistics. You can reframe some
problems in logistics into networks of queues. In this post, I'll provide a
simple example of modeling an airplane deplaning and boarding at a gate.

To keep things sane, we'll have N planes launched from some airport somethere,
either arriving or waiting to arrive at a single gate, about once every 110
minutes, distributed exponentially. To make it slightly more realistic, we'll
have two simultaneous services: passengers deplaning and boarding, and ground
crews pulling bags off, and putting bags into the hold. These take about 45
minutes, distributed normally, with different standard deviations.

{% highlight python %}
import simpy
import numpy

def generate_interarrival():
    return numpy.random.exponential(110./1) # One arrival per 110 minutes

def generate_deplane_board_service():
    return numpy.random.normal(45, 5) # 45 minute turnaround

def generate_flight_preparation_service():
    return numpy.random.normal(45, 3) # 45 minute turnaround

def airport(env, gates):
    i = 0
    while True:
        i += 1
        yield env.timeout(generate_interarrival())
        env.process(airplane(env, i, gates))

def airplane(env, airplane_number, gates):
    with gates.request() as request:
        t_arrival = env.now
        print(f'{env.now:.1f}', f'airplane {airplane_number} arrives')
        yield request
        t_deplane_board = env.now
        print(f'{env.now:.1f}', f'airplane {airplane_number} deplaning/boarding')
        yield env.timeout(max(generate_flight_preparation_service(), generate_deplane_board_service()))
        t_depart = env.now
        print(f'  {t_depart - t_deplane_board:.1f} minutes on tarmac for airplane {airplane_number}')
        print(f'{env.now:.1f}', f'airplane {airplane_number} departs')
        wait_t.append(t_deplane_board - t_arrival)
        process_t.append(t_depart - t_deplane_board)
        

wait_t = []
process_t = []

env = simpy.Environment()
gates = simpy.Resource(env, capacity=1)
env.process(airport(env, gates))
env.run(until=720)
{% endhighlight %}

Okay, that was kind of a lot. The output is kind of what you would expect.

{% highlight console %}
126.2 min., airplane 1 arrives
126.2 min., airplane 1 deplaning/boarding
131.1 min., airplane 2 arrives
  49.7 min. on tarmac for airplane 1
175.9 min., airplane 1 departs
175.9 min., airplane 2 deplaning/boarding
  45.9 min. on tarmac for airplane 2
221.7 min., airplane 2 departs
234.3 min., airplane 3 arrives
234.3 min., airplane 3 deplaning/boarding
  43.7 min. on tarmac for airplane 3
278.0 min., airplane 3 departs
560.0 min., airplane 4 arrives
560.0 min., airplane 4 deplaning/boarding
605.2 min., airplane 5 arrives
  50.0 min. on tarmac for airplane 4
610.0 min., airplane 4 departs
610.0 min., airplane 5 deplaning/boarding
  47.9 min. on tarmac for airplane 5
657.9 min., airplane 5 departs
710.2 min., airplane 6 arrives
710.2 min., airplane 6 deplaning/boarding
{% endhighlight %}

And then we can plot the waiting times as,

{% highlight python %}
plt.step([1,2,3,4,5,], process_t, where='mid')
plt.title('Waiting time on tarmac')
plt.savefig('waiting-times.png', dpi=200)
{% endhighlight %}

![Waiting Times on Tarmac](/assets/images/waiting-times.png)
