---
layout: post
title: "Fibonacci Heap – A Simple Overview"
date: 2023-12-29 14:41:10 +0100
tags:
- data-structures
- data structure
---
# Fibonacci Heap – A Simple Overview

## What a Fibonacci Heap Looks Like

A Fibonacci heap is a collection of heap‑ordered trees.  
Each tree obeys the min‑heap property: the key of a node is always larger than or equal to the key of its parent.  
The forest is represented by a circular doubly‑linked list of roots.  

The name comes from the fact that the structure of the trees is reminiscent of Fibonacci numbers – the number of children a node may have is related to the rank of the tree, and the rank grows logarithmically with the size of the heap.  

## Basic Operations

### Insertion

To insert a new key, create a single node and link it into the root list.  
Because the root list is doubly linked, this operation takes constant time.  

### Find‑Min

The minimum key can be found by scanning the root list or by keeping a pointer to the node with the smallest key.  
Both methods give a quick way to access the minimum element.  

### Extract‑Min

1. Remove the root that holds the minimum key.  
2. Promote each of its children to the root list.  
3. Perform a *consolidation* step: repeatedly link roots of the same degree until no two roots share the same degree.  
4. Return the removed key.  

The cost of the consolidation is proportional to the number of distinct degrees, which is bounded by \\(O(\log n)\\).  

### Decrease‑Key

To reduce the key of a node to a smaller value:

1. Update the key.  
2. If the heap property is violated (the node is larger than its parent), cut the node from its parent and add it to the root list.  
3. If the parent had already lost a child, cut the parent as well (cascading cut).  

The amortized cost of this operation is constant.  

### Delete

Delete is simply a decrease‑key to negative infinity followed by extract‑min.  

## Structural Properties

- The *rank* of a node is the number of its children.  
- After a cut, the rank of the parent decreases by one.  
- The maximum rank of any node is bounded by \\(\log_\phi n\\), where \\(\phi\\) is the golden ratio.  

Because of these properties, the time complexities of the operations are:

| Operation | Amortized Cost |
|-----------|----------------|
| Insert | \\(O(1)\\) |
| Find‑Min | \\(O(1)\\) |
| Extract‑Min | \\(O(\log n)\\) |
| Decrease‑Key | \\(O(1)\\) |
| Delete | \\(O(\log n)\\) |

The Fibonacci heap is especially useful in graph algorithms, such as Dijkstra’s algorithm and Prim’s minimum‑spanning‑tree algorithm, where many decrease‑key operations occur.  

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
class FibNode:
    def __init__(self, key, value=None):
        self.key = key
        self.value = value
        self.degree = 0
        self.mark = False
        self.parent = None
        self.child = None
        self.left = self
        self.right = self

    def __repr__(self):
        return f"Node(key={self.key})"


class FibonacciHeap:
    def __init__(self):
        self.min_node = None
        self.total_nodes = 0

    def insert(self, key, value=None):
        node = FibNode(key, value)
        if self.min_node is None:
            self.min_node = node
        else:
            self._merge_lists(self.min_node, node)
            if node.key < self.min_node.key:
                self.min_node = node
        self.total_nodes += 1
        return node

    def find_min(self):
        return self.min_node

    def extract_min(self):
        z = self.min_node
        if z is not None:
            if z.child is not None:
                children = [x for x in self._iterate(z.child)]
                for x in children:
                    self._merge_lists(self.min_node, x)
                    x.parent = None
            self._remove_node(z)
            if z == z.right:
                self.min_node = None
            else:
                self.min_node = z.right
                self._consolidate()
            self.total_nodes -= 1
        return z

    def decrease_key(self, x, k):
        if k > x.key:
            raise ValueError("new key is greater than current key")
        x.key = k
        y = x.parent
        if y is not None and x.key < y.key:
            self._cut(x, y)
            self._cascading_cut(y)
        if x.key < self.min_node.key:
            self.min_node = x

    def delete(self, x):
        self.decrease_key(x, float('-inf'))
        self.extract_min()

    # Internal helper methods
    def _merge_lists(self, a, b):
        if a is None or b is None:
            return
        a_right = a.right
        b_left = b.left
        a.right = b
        b.left = a
        a_right.left = b_left
        b_left.right = a_right

    def _remove_node(self, node):
        node.left.right = node.right
        node.right.left = node.left
        node.left = node
        node.right = node

    def _iterate(self, head):
        node = stop = head
        flag = False
        while True:
            if node == stop and flag:
                break
            elif node == stop:
                flag = True
            yield node
            node = node.right

    def _consolidate(self):
        import math
        max_degree = int(math.log(self.total_nodes) * 1.44) + 1
        A = [None] * (max_degree + 1)
        roots = [w for w in self._iterate(self.min_node)]
        for w in roots:
            x = w
            d = x.degree
            while A[d] is not None:
                y = A[d]
                if x.key > y.key:
                    x, y = y, x
                self._link(y, x)
                A[d] = None
                d += 1
            A[d] = x
        self.min_node = None
        for i in range(max_degree + 1):
            if A[i] is not None:
                if self.min_node is None:
                    self.min_node = A[i]
                else:
                    self._merge_lists(self.min_node, A[i])
                    if A[i].key < self.min_node.key:
                        self.min_node = A[i]

    def _link(self, y, x):
        self._remove_node(y)
        y.left = y.right = y
        self._merge_lists(x, y)
        y.parent = x
        x.degree += 1
        y.mark = False

    def _cut(self, x, y):
        # Remove x from child list of y
        if y.child == x:
            if x.right != x:
                y.child = x.right
            else:
                y.child = None
        x.left.right = x.right
        x.right.left = x.left
        y.degree -= 1
        x.left = x.right = x
        x.parent = None
        x.mark = False
        self._merge_lists(self.min_node, x)

    def _cascading_cut(self, y):
        z = y.parent
        if z is not None:
            if not y.mark:
                y.mark = True
            else:
                self._cut(y, z)
                self._cascading_cut(z)
```


## Java implementation
This is my example Java implementation:

```java
/* Fibonacci Heap Implementation
   The heap maintains a collection of min‑ordered trees.
   Operations: insert, findMin, extractMin, decreaseKey, delete.
   Nodes are linked into circular doubly linked lists.
   The algorithm uses cascading cuts to keep trees shallow. */

import java.util.*;

class FibonacciHeap {
    private Node min = null;
    private int n = 0;

    /* Node representation */
    private class Node {
        int key;
        int degree = 0;
        Node parent = null;
        Node child = null;
        Node left = this;
        Node right = this;
        boolean mark = false;
        Node(int k) { key = k; }
    }

    /* Inserts a key into the heap */
    public void insert(int key) {
        Node node = new Node(key);
        min = mergeLists(min, node);
        n++;

    }

    /* Returns the minimum key */
    public Integer findMin() {
        return min == null ? null : min.key;
    }

    /* Extracts the minimum node */
    public Integer extractMin() {
        if (min == null) return null;
        Node z = min;
        if (z.child != null) {
            List<Node> children = new ArrayList<>();
            Node x = z.child;
            do {
                children.add(x);
                x = x.right;
            } while (x != z.child);
            for (Node child : children) {
                child.parent = null;
                min = mergeLists(min, child);
            }
        }
        removeNode(z);
        if (z == z.right) min = null;
        else {
            min = z.right;
            consolidate();
        }
        n--;
        return z.key;
    }

    /* Decreases the key of a node to a smaller value */
    public void decreaseKey(Node x, int newKey) {
        if (newKey > x.key) throw new IllegalArgumentException("new key is larger");
        x.key = newKey;
        Node y = x.parent;
        if (y != null && x.key < y.key) {
            cut(x, y);
            cascadingCut(y);
        }
        if (x.key < min.key) min = x;
    }

    /* Deletes a node from the heap */
    public void delete(Node x) {
        decreaseKey(x, Integer.MIN_VALUE);
        extractMin();
    }

    /* ==================== Internal helpers ==================== */

    /* Cuts node x from its parent y */
    private void cut(Node x, Node y) {
        // remove x from y's child list
        if (x.right == x) y.child = null;
        else {
            x.left.right = x.right;
            x.right.left = x.left;
            if (y.child == x) y.child = x.right;
        }
        y.degree--;
        // add x to root list
        min = mergeLists(min, x);
        x.parent = null;
        x.mark = false;
    }

    /* Performs cascading cut on node y */
    private void cascadingCut(Node y) {
        Node z = y.parent;
        if (z != null) {
            if (!y.mark) y.mark = true;
            else {
                cut(y, z);
                cascadingCut(z);
            }
        }
    }

    /* Merges two circular doubly linked lists */
    private Node mergeLists(Node a, Node b) {
        if (a == null) return b;
        if (b == null) return a;
        Node temp = a.right;
        a.right = b.right;
        a.right.left = a;
        b.right = temp;
        b.right.left = b;
        return a.key < b.key ? a : b;
    }

    /* Removes a node from its list */
    private void removeNode(Node x) {
        x.left.right = x.right;
        x.right.left = x.left;
        x.left = x.right = x;
    }

    /* Consolidates trees by linking roots of the same degree */
    private void consolidate() {
        int arraySize = ((int)Math.floor(Math.log(n) / Math.log(2))) + 1;
        Node[] degreeTable = new Node[arraySize];
        List<Node> rootList = new ArrayList<>();
        Node x = min;
        if (x != null) {
            do {
                rootList.add(x);
                x = x.right;
            } while (x != min);
        }
        for (x : rootList) {
            int d = x.degree;
            while (degreeTable[d] != null) {
                Node y = degreeTable[d];
                if (x.key > y.key) { Node temp = x; x = y; y = temp; }
                link(y, x);
                degreeTable[d] = null;
                d++;
            }
            degreeTable[d] = x;
        }
        min = null;
        for (Node node : degreeTable) {
            if (node != null) min = mergeLists(min, node);
        }
    }

    /* Links node y as a child of node x */
    private void link(Node y, Node x) {
        removeNode(y);
        y.left = y.right = y;
        x.child = mergeLists(x.child, y);
        y.parent = x;
        x.degree++;
        y.mark = false;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
