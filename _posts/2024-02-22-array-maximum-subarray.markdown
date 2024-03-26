---
layout: post
title: 'Array - Maximum Subarray'
date: 2024-02-22 00:00:00 -0700
tags: algorithms
---

This is an implementation of [Kadane's](https://en.wikipedia.org/wiki/Joseph_Born_Kadane)
[Algorithm](https://en.wikipedia.org/wiki/Maximum_subarray_problem).

{% highlight python %}
class Array:

    def __init__(self, data: list):
        self.data = data

    def maximum_subarray_sum(self):
        if len(self.data) == 0:
            return None
        if len(self.data) == 1:
            return self.data[0]
        best_sum = float('-inf')
        current_sum = 0
        for i, item in enumerate(self.data):
            current_sum = max(item, current_sum + item)
            best_sum = max(best_sum, current_sum)
        return best_sum

    def maximum_subarray_elements(self):
        if len(self.data) == 0:
            return None
        if len(self.data) == 1:
            return self.data
        best_sum = float('-inf')
        current_sum = 0
        begin_index = 0
        end_index = 0
        for i, item in enumerate(self.data):
            if item > current_sum + item:
                current_sum = item
                begin_index = i
                end_index = i
            else:
                current_sum += item
            # print(f'  current: {current_sum}')
            if current_sum > best_sum:
                best_sum = current_sum
                end_index = i
            else:
                pass
        result = self.data[begin_index:end_index+1]
        return result

assert Array([-2,1,-3,4,-1,2,1,-5,4]).maximum_subarray_sum() == 6
assert Array([5,4,-1,7,8]).maximum_subarray_sum() == 23
assert Array([-1,8,8]).maximum_subarray_sum() == 16
assert Array([-1,8,-1]).maximum_subarray_sum() == 8
assert Array([-1]).maximum_subarray_sum() == -1
assert Array([-1, -2, -3]).maximum_subarray_sum() == -1
assert Array([]).maximum_subarray_sum() == None

assert Array([-2,1,-3,4,-1,2,1,-5,4]).maximum_subarray_elements() == [4, -1, 2, 1]
assert Array([5,4,-1,7,8]).maximum_subarray_elements() == [5, 4, -1, 7, 8]
assert Array([-2, -5, 6, -2, -3, 1, 5, -6]).maximum_subarray_elements() == [6, -2, -3, 1, 5]
assert Array([-3, -4, 5, -1, 2, -4, 6, -1]).maximum_subarray_elements() == [5, -1, 2, -4, 6]
assert Array([-2, 3, -1, 2]).maximum_subarray_elements() == [3, -1, 2]
{% endhighlight %}
