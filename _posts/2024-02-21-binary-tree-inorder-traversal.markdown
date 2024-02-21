---
layout: post
title:  "Binary Tree - In-Order Traversal"
date:   2024-02-21 00:00:00 -0700
categories: python
---

Most explanations of binary tree in-order traversal just cover the recursive
implementation. I've provided an iterative solution also.

{% highlight python %}
class Stack:

    def __init__(self):
        self.data = []

    def push(self, item):
        self.data.append(item)

    def pop(self):
        return self.data.pop()

    def peek(self):
        return self.data[-1]

    def __len__(self):
        return len(self.data)

    def __repr__(self):
        return str(self.data)
        
class BinaryTreeNode:

    def __init__(self, parent, data):
        self.data = data
        self.left = None
        self.right = None

    def __repr__(self):
        return f'{self.data}'

class BinaryTree:

    def __init__(self):
        self.root = None
        self.result = []

    def add(self, node, side, data):
        new_node = BinaryTreeNode(node, data)
        if side == 'left':
            node.left = new_node
        elif side == 'right':
            node.right = new_node
        elif side == 'root':
            self.root = new_node
        return new_node

    def visit(self, node):
        self.result.append(node.data)
        print(f'visit {node.data}')

    def iterative_inorder_traversal(self):
        node = self.root
        stack = Stack()
        stack.push(node)
        state = 'PUSH'
        while True:
            if len(stack) == 0:
                break
            if state == 'PUSH':
                node = stack.peek()
                if node.left:
                    stack.push(node.left)
                else:
                    state = 'POP'
            elif state == 'POP':
                node = stack.pop()
                self.visit(node)
                if node.right:
                    stack.push(node.right)
                    state = 'PUSH'

    def recursive_inorder_traversal(self, node):
        if node is None:
            return
        self.recursive_inorder_traversal(node.left)
        self.visit(node)
        self.recursive_inorder_traversal(node.right)
{% endhighlight %}

Here is some test code to test the implementation.

{% highlight python %}
t = BinaryTree()
root = t.add(None, 'root', 1)
n2 = t.add(root, 'left', 2)
n3 = t.add(root, 'right', 3)
t.add(n2, 'left', 4)
t.add(n2, 'right', 5)
t.add(n3, 'left', 6)
t.add(n3, 'right', 7)
t.iterative_inorder_traversal()
assert t.result == [4, 2, 5, 1, 6, 3, 7]
print()

t = BinaryTree()
root = t.add(None, 'root', 1)
n2 = t.add(root, 'left', 2)
n3 = t.add(root, 'right', 3)
t.add(n2, 'left', 4)
n5 = t.add(n3, 'left', 5)
t.add(n3, 'right', 6)
t.add(n5, 'left', 7)
t.add(n5, 'right', 8)
t.iterative_inorder_traversal()
assert t.result == [4, 2, 1, 7, 5, 8, 3, 6]
print()

t = BinaryTree()
root = t.add(None, 'root', 1)
n2 = t.add(root, 'right', 2)
t.add(n2, 'left', 3)
t.iterative_inorder_traversal()
assert t.result == [1, 3, 2]
print()

t = BinaryTree()
root = t.add(None, 'root', 1)
n2 = t.add(root, 'right', 2)
t.add(n2, 'left', 3)
t.recursive_inorder_traversal(t.root)
assert t.result == [1, 3, 2]
{% endhighlight %}

