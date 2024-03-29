---
layout: post
title: 'Trie - Lexicographic Sort'
date: 2024-02-13 19:00:00 -0700
tags: data-structures
---

This performs lexicographic sorting using a Trie iteratively, without
recursion. This involves a slight hack: knowing that the backtick character
occurs just before "a" in the ASCII table.

The part I find interesting abou this is the `visited` attribute I added
to the `TrieNode`. When the `lexicographic_sort` method runs, it toggles the
`visited` attribute on each node using a "plus one, modulo two" operation.
This way, it doesn't matter what the actual value of the `visited`
attribute is, and we don't need to worry about "un-setting" it, we just check
whether the child's `visited` attribute is the same as its parent, or not.

{% highlight python %}
class TrieNode:

    def __init__(self, item: str):
        self.item = item
        self.parent = None
        self.children = [None] * 27
        self.visited = 0

    def __repr__(self):
        return self.item

class Trie:

    def __init__(self):
        self.root = TrieNode('')
        self.visited = 0

    def insert(self, item: str):
        item += '`' # <-- hack
        item = item.lower()
        i = 0
        node = self.root
        while i < len(item) and node.children[ord(item[i]) - ord('`')]:
            lexical_index = ord(item[i]) - ord('`')
            node = node.children[lexical_index]
            i += 1
        while i < len(item):
            lexical_index = ord(item[i]) - ord('`')
            new_node = TrieNode(item[i])
            new_node.parent = node
            node.children[lexical_index] = new_node
            node = new_node
            i += 1

    def lexicographic_sort(self):
        self.visited = (self.visited + 1) % 2
        result = []
        ptr = self.root
        word = ''
        while True:
            if ptr is None:
                break
            if any(ptr.children):
                visited = []
                for child in ptr.children:
                    if child is None:
                        continue
                    if child.visited == self.visited:
                        visited.append(True)
                    else:
                        visited.append(False)
                        word += child.item
                        child.visited = (child.visited + 1) % 2
                        ptr = child
                        break
                if all(visited):
                    ptr = ptr.parent
                    word = word[:-1]
                    continue
            else:
                result.append(word[:-1])
                word = word[:-1]
                ptr = ptr.parent
        return result

t = Trie()
words = [
'lexicographic', 'sorting', 'of', 'a', 'set', 'of', 'keys', 'can', 'be',
'accomplished', 'with', 'a', 'simple', 'trie', 'based', 'algorithm',
'we', 'insert', 'all', 'keys', 'in', 'a', 'trie', 'output', 'all',
'keys', 'in', 'the', 'trie', 'by', 'means', 'of', 'preorder',
'traversal', 'which', 'results', 'in', 'output', 'that', 'is', 'in',
'lexicographically', 'increasing', 'order', 'preorder', 'traversal',
'is', 'a', 'kind', 'of', 'depth', 'first', 'traversal'
]
for word in words:
t.insert(word)
assert t.lexicographic_sort() == sorted(set(words))
{% endhighlight %}
