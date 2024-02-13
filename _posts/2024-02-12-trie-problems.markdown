---
layout: post
title:  "Trie Problems"
date:   2024-02-12 19:00:00 -0700
categories: data-structures
---

I'd like to collect a few programming problems related to Tries here.

The basic data structure for a Trie in Python is this.

{% highlight python %}
class TrieNode:

    def __init__(self, item: str):
        self.item = item
        self.children = {}

    def __repr__(self):
        return self.item

class Trie:

    def __init__(self):
        self.root = TrieNode('')

    def insert(self, item: str):
        i = 0
        node = self.root
        while i < len(item) and item[i] in list(node.children.keys()):
            node = node.children.get(item[i])
            i += 1
        while i < len(item):
            new_node = TrieNode(item[i])
            node.children[item[i]] = new_node
            node = new_node
            i += 1
{% endhighlight %}

To calcualte the longest common prefix (LCP) then you can add this method to the Trie class.

{% highlight python %}
def longest_common_prefix(self) -> list:
    lcp = []
    i = 0
    node = self.root
    while True:
        children = list(node.children.keys())
        if len(children) == 1:
            key = children[0]
            lcp.append(key)
            node = node.children.get(key)
        else:
            return lcp
{% endhighlight %}

And then test this method like this.

{% highlight python %}
t = Trie()
t.insert("indigo")
t.insert("indie")
assert t.longest_common_prefix() == ['i', 'n', 'd', 'i']
{% endhighlight %}
