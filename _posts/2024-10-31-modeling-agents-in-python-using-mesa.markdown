---
layout: post
title: "Modeling Agents in Python Using Mesa"
date: 2024-10-30 00:00:00 -0700
tags: python
---

In school, we learned how to write systems of differential equations to model
predator/prey relationships in (abstract) natural habitats. The problem with
this was that its weird to talk about 2.5 coyotes or rabbits, and adding more
types of entities gets very ugly, very quickly. Agent based models allow you
to trade systems of differential equations, for code, and worrying about string
encodings, timezones, rounding errors, and in short, chaos.

The Mesa module allows you to create agent based models. These allow you to
break a problem into entities, and then define how those entities react with
each other. You might use this to model entities fighting for resources, or
working together. I'll start with a simple model that uses a graph, where an
agent, Pacer, paces back and forth on a two node graph, printing its location
at each step.

{% highlight python %}
from mesa import Agent, Model
from mesa.time import RandomActivationByType
from mesa.space import NetworkGrid
import networkx as nx

class PacerAgent(Agent):

    def __init__(self, unique_id, model):
        super().__init__(unique_id, model)
        self.pos = None

    def step(self):
        print(self.pos)

        if self.pos == 0:
            next_pos = self.model.grid.get_neighborhood(self.pos)[0]
            self.model.grid.move_agent(self, next_pos)

        elif self.pos == 1:
            next_pos = self.model.grid.get_neighborhood(self.pos)[0]
            self.model.grid.move_agent(self, next_pos)

class LogisticsModel(Model):

    def __init__(self, num_pacers):
        super().__init__()
        self.schedule = RandomActivationByType(self)

        G = nx.Graph()
        G.add_nodes_from([0, 1])
        G.add_edges_from([(0, 1)])
        self.grid = NetworkGrid(G)

        for i in range(num_pacers):
            pacer = PacerAgent(i, self)
            self.schedule.add(pacer)
            self.grid.place_agent(pacer, 0)

    def step(self):
        self.schedule.step()

model = LogisticsModel(1)
for i in range(10):
    model.step()
{% endhighlight %}

That should produce this:

{% highlight console %}
0
1
0
1
0
1
0
1
0
1
{% endhighlight %}

Great, that works. These are the lessons I learned:

  - `self.pos` is a member of Agent; renaming this `position` will break the code
  - `networkx.Graph` nodes need to be integers to work with `mesa.space.NetworkGrid`, not strings
  - `mesa.space.NetworkGrid.get_neighbors()` returns `mesa.Agent` objects, while `mesa.space.NetworkGrid.get_neighborhood()` returns `networkx.Node` objects

Here is a more complicated example. There are two, differently sized Trucks,
carrying boxes from one warehouse to another, across a simple two node grid.


{% highlight python %}
from mesa import Agent, Model
from mesa.time import RandomActivationByType
from mesa.space import NetworkGrid
import networkx as nx

class WarehouseAgent(Agent):
    
    def __init__(self, unique_id, model, stock, capacity):
        super().__init__(unique_id, model)
        self.capacity = capacity
        self.stock = stock

class TruckAgent(Agent):
    
    def __init__(self, unique_id, model, capacity):
        super().__init__(unique_id, model)
        self.position = None
        self.capacity = capacity
        self.load = 0

    def step(self):
        print(f'Truck {self.unique_id} with load {self.load} at position {self.pos} with stock A:{self.model.warehouse_A.stock} B:{self.model.warehouse_B.stock}')

        # If the truck is at warehouse A:
        if self.pos == 0:
            if self.load == self.capacity:
                next_position = self.model.grid.get_neighborhood(self.pos)[0]
                self.model.grid.move_agent(self, next_position)
            else:
                if self.model.warehouse_A.stock > 0:
                    self.model.warehouse_A.stock -= 1
                    self.load += 1
                else:
                    pass

        # If the truck is at warehouse B:
        elif self.pos == 1:
            if self.load == 0:
                next_position = self.model.grid.get_neighborhood(self.pos)[0]
                self.model.grid.move_agent(self, next_position)
            else:
                self.load -= 1
                self.model.warehouse_B.stock += 1

class LogisticsModel(Model):
    
    def __init__(self):
        super().__init__()
        self.schedule = RandomActivationByType(self)

        G = nx.Graph()
        G.add_nodes_from([0, 1])
        G.add_edges_from([(0, 1)])
        self.grid = NetworkGrid(G)

        self.warehouse_A = WarehouseAgent(0, self, 10, 10)
        self.warehouse_B = WarehouseAgent(1, self, 0, 10)

        # place warehouse agents on the grid
        self.grid.place_agent(self.warehouse_A, 0)
        self.grid.place_agent(self.warehouse_B, 1)

        # create and place truck agents
        truck = TruckAgent(0, self, capacity=2)
        self.schedule.add(truck)
        self.grid.place_agent(truck, 0)

        truck = TruckAgent(1, self, capacity=3)
        self.schedule.add(truck)
        self.grid.place_agent(truck, 0)

    def step(self):
        self.schedule.step()

model = LogisticsModel()
while model.warehouse_B.stock < 10
    model.step()
{% endhighlight %}

And here is the output:

{% highlight console %}
Truck 0 with load 0 at position 0 with stock A:10 B:0
Truck 1 with load 0 at position 0 with stock A:9 B:0
Truck 1 with load 1 at position 0 with stock A:8 B:0
Truck 0 with load 1 at position 0 with stock A:7 B:0
Truck 1 with load 2 at position 0 with stock A:6 B:0
Truck 0 with load 2 at position 0 with stock A:5 B:0
Truck 0 with load 2 at position 1 with stock A:5 B:0
Truck 1 with load 3 at position 0 with stock A:5 B:1
Truck 0 with load 1 at position 1 with stock A:5 B:1
Truck 1 with load 3 at position 1 with stock A:5 B:2
Truck 0 with load 0 at position 1 with stock A:5 B:3
Truck 1 with load 2 at position 1 with stock A:5 B:3
Truck 1 with load 1 at position 1 with stock A:5 B:4
Truck 0 with load 0 at position 0 with stock A:5 B:5
Truck 1 with load 0 at position 1 with stock A:4 B:5
Truck 0 with load 1 at position 0 with stock A:4 B:5
Truck 1 with load 0 at position 0 with stock A:3 B:5
Truck 0 with load 2 at position 0 with stock A:2 B:5
Truck 1 with load 1 at position 0 with stock A:2 B:5
Truck 0 with load 2 at position 1 with stock A:1 B:5
Truck 1 with load 2 at position 0 with stock A:1 B:6
Truck 0 with load 1 at position 1 with stock A:0 B:6
Truck 1 with load 3 at position 0 with stock A:0 B:7
Truck 0 with load 0 at position 1 with stock A:0 B:7
Truck 1 with load 3 at position 1 with stock A:0 B:7
Truck 0 with load 0 at position 0 with stock A:0 B:8
Truck 1 with load 2 at position 1 with stock A:0 B:8
Truck 0 with load 0 at position 0 with stock A:0 B:9
Truck 1 with load 1 at position 1 with stock A:0 B:9
Truck 0 with load 0 at position 0 with stock A:0 B:10
{% endhighlight %}