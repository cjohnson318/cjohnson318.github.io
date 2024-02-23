---
layout: post
title:  "Array - Smallest k-Element Set"
date:   2024-02-23 00:00:00 -0700
categories: python
mathjax: true
---

These two algorithms do the same thing, but the "moving queue" solution is
much more efficient. The reason is that the "moving queue" solutions iterates
over the data once, while the "brute force" solution iterates over $N*(N-1)$
subsets of the data, which is $O(N^2)$, where $N$ is the size of the data.

{% highlight python %}
import itertools
import random

class Array:

    def __init__(self, data):
        self.data = data

    def k_element_subarrays_brute_force(self, k):
        N = len(self.data)
        indices = list(range(N+1))
        result = float('inf')
        for item in itertools.combinations(indices, 2):
            i, j = item
            subarr = self.data[i:j]
            if len(set(subarr)) < k:
                continue
            if len(set(subarr)) == k:
                if len(subarr) < result:
                    result = len(subarr)
        if result == float('inf'):
            result = -1
        return result

    def k_element_subarrays_moving_queue(self, k):
        N = len(self.data)
        result = float('inf')
        kvs = {}
        queue = []
        arr = None
        for i, item in enumerate(self.data):
            if len(queue) == k:
                indices = [kvs[it] for it in queue]
                _min, _max = min(indices), max(indices)
                n = _max - _min + 1
                if n < result:
                    # arr = self.data[_min:_max+1]
                    result = n
            kvs[item] = i
            if len(queue) <= k and item not in queue:
                queue.append(item)
            if len(queue) > k:
                queue.pop(0)

        if result == float('inf'):
            result = -1
        return result

for i in range(1000):
    r = [random.randint(-5,5) for _ in range(100)]
    assert Array(r).k_element_subarrays_brute_force(3) == Array(r).k_element_subarrays_moving_queue(3), r
{% endhighlight %}

