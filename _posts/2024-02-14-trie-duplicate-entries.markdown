---
layout: post
title: 'Trie - Duplicate Entries'
date: 2024-02-14 19:00:00 -0700
tags: data-structures
---

This ended up being pretty easy. Just add an optional attribute on the TrieNode
that informs the next insert whether a particular string has been inserted
before, and then update a list of duplicated on the Trie object.

{% highlight python %}
class TrieNode:

    def __init__(self, item:str, leaf:int=None):
        self.item = item
        self.children = {}
        self.leaf = leaf

    def __repr__(self):
        return self.item

class Trie:

    def __init__(self):
        self.root = TrieNode('')
        self.count = 0
        self.duplicates = []

    def insert(self, item: str):
        self.count += 1
        i = 0
        node = self.root
        while i < len(item) and item[i] in list(node.children.keys()):
            node = node.children.get(item[i])
            if i == len(item) - 1 and node.leaf is not None:
                self.duplicates.append(self.count)
            i += 1
        while i < len(item):
            if i == len(item) - 1:
                new_node = TrieNode(item[i], self.count)
            else:
                new_node = TrieNode(item[i])
            node.children[item[i]] = new_node
            node = new_node
            i += 1

mat = [
[0, 1, 1, 0, 0],
[1, 0, 0, 1, 0],
[1, 0, 0, 1, 0],
[0, 0, 1, 1, 0],
[0, 1, 1, 0, 0]
]

t = Trie()
for row in mat:
t.insert(row)
assert t.duplicates == [3, 5]
{% endhighlight %}
