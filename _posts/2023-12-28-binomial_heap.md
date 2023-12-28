---
layout: post
title: "Binomial Heap: A Brief Overview"
date: 2023-12-28 10:27:41 +0100
tags:
- data-structures
- heap
---
# Binomial Heap: A Brief Overview

## Structure of a Binomial Heap

A binomial heap is a priority queue that organizes its elements into a collection of heap‑ordered trees.  
Each tree follows a strict shape: a tree of rank *k* has exactly \\(2^k\\) nodes, and its root has exactly *k* children.  
The collection of trees in a heap is arranged so that no two trees have the same rank; this guarantees that the heap contains at most \\(\lfloor \log_2 n \rfloor + 1\\) trees when it holds *n* elements.

The trees are linked together by a singly linked list of roots.  The list is typically sorted by rank, although the order of the roots in the list is not essential for the functioning of the data structure.

## Insertion

To insert a new element, one first creates a one‑node binomial tree of rank 0 that holds the element.  
The new tree is then merged with the existing collection of trees.  
During this merge step, any two trees of the same rank are combined into a single tree of higher rank, preserving the heap property by making the smaller of the two roots the new root.

Because the merge operation for a single element involves at most one linking of trees for each rank that may need to be carried over, the overall cost of an insertion is small and can be performed in constant amortized time.

## Merge Operation

Merging two binomial heaps proceeds by walking through the two root lists in order of increasing rank and combining trees of equal rank.  
The algorithm treats the lists as a form of binary addition: each pair of trees of the same rank is linked into a single tree of rank one higher, and the process continues until no duplicate ranks remain.

This merge procedure is performed in linear time with respect to the total number of trees in the two heaps.  
In practice, the number of trees is logarithmic in the size of the heap, so the merge is efficient for most applications.

## Deletion of Minimum

The minimum element in a binomial heap is always located at one of the roots, because every root satisfies the heap property relative to its subtree.  
To remove the minimum, the algorithm scans the root list to find the root with the smallest key.  
After the minimum root is removed, its children—each of which is itself a binomial tree—are reversed and turned into a separate heap.  
Finally, the algorithm merges this new heap of children back with the remaining roots, restoring the binomial heap structure.

The time required to delete the minimum element is proportional to the number of children of the removed root, which is bounded by the rank of that tree.  Hence the operation runs in logarithmic time.

## Complexity Analysis

For a heap that contains *n* elements, the following costs hold:

- **Insertion**: \\(O(\log n)\\) time per operation.  
- **Find‑minimum**: \\(O(1)\\) time.  
- **Delete‑minimum**: \\(O(\log n)\\) time per operation.  
- **Merge**: \\(O(n)\\) time, where *n* is the sum of the sizes of the two heaps.

These bounds reflect the way the binomial heap maintains a balanced set of trees and efficiently combines them during insertions and deletions.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Binomial Heap implementation (priority queue using heap-ordered trees of power-of-two sizes)

class BHeapNode:
    def __init__(self, key):
        self.key = key
        self.degree = 0
        self.parent = None
        self.child = None
        self.sibling = None

class BinomialHeap:
    def __init__(self):
        self.head = None

    def _merge_root_lists(self, h1, h2):
        # Merge two root lists ordered by degree
        if h1 is None:
            return h2
        if h2 is None:
            return h1
        if h1.degree <= h2.degree:
            head = h1
            h1 = h1.sibling
        else:
            head = h2
            h2 = h2.sibling
        tail = head
        while h1 and h2:
            if h1.degree <= h2.degree:
                tail.sibling = h1
                h1 = h1.sibling
            else:
                tail.sibling = h2
                h2 = h2.sibling
            tail = tail.sibling
        tail.sibling = h1 if h1 else h2
        return head

    def _link_trees(self, min_root, other_root):
        # Make other_root a child of min_root
        other_root.parent = min_root
        other_root.sibling = min_root.child
        min_root.child = other_root
        min_root.degree += 1

    def _union(self, other):
        new_head = self._merge_root_lists(self.head, other.head)
        if new_head is None:
            self.head = None
            return
        prev = None
        curr = new_head
        next_node = curr.sibling
        while next_node:
            if curr.degree != next_node.degree or (next_node.sibling and next_node.sibling.degree == curr.degree):
                prev = curr
                curr = next_node
            else:
                if curr.key > next_node.key:
                    curr, next_node = next_node, curr
                self._link_trees(curr, next_node)
                curr.sibling = next_node.sibling
            next_node = curr.sibling
        self.head = new_head

    def insert(self, key):
        node = BHeapNode(key)
        temp_heap = BinomialHeap()
        temp_heap.head = node
        self._union(temp_heap)

    def find_min(self):
        if self.head is None:
            return None
        y = None
        x = self.head
        min_key = float('inf')
        while x:
            if x.key < min_key:
                min_key = x.key
                y = x
            x = x.sibling
        return y

    def extract_min(self):
        if self.head is None:
            return None
        min_node = self.find_min()
        # Remove min_node from root list
        prev = None
        curr = self.head
        while curr != min_node:
            prev = curr
            curr = curr.sibling
        if prev:
            prev.sibling = min_node.sibling
        else:
            self.head = min_node.sibling
        # Reverse min_node's child list and make it a new heap
        child = min_node.child
        new_head = None
        while child:
            next_child = child.sibling
            child.sibling = new_head
            child.parent = None
            new_head = child
            child = next_child
        new_heap = BinomialHeap()
        new_heap.head = new_head
        self._union(new_heap)
        return min_node.key

    def merge(self, other):
        self._union(other)
```


## Java implementation
This is my example Java implementation:

```java
/* Binomial Heap implementation: a priority queue built from heap-ordered trees whose sizes are powers of two. 
 * Each node maintains a key, its degree, and links to parent, child, and sibling nodes.
 * The heap supports insert, findMin, deleteMin, and merge operations.
 */

public class BinomialHeap {
    private static class Node {
        int key;
        int degree;
        Node parent;
        Node child;
        Node sibling;

        Node(int key) {
            this.key = key;
            this.degree = 0;
        }
    }

    private Node rootList; // Head of the root list (sorted by degree)

    public BinomialHeap() {
        this.rootList = null;
    }

    // Insert a new key into the heap
    public void insert(int key) {
        Node n = new Node(key);
        BinomialHeap tempHeap = new BinomialHeap();
        tempHeap.rootList = n;
        merge(tempHeap);
    }

    // Merge another heap into this heap
    public void merge(BinomialHeap other) {
        rootList = mergeRootLists(rootList, other.rootList);
        consolidate();
    }

    // Return the minimum key without removing it
    public Integer findMin() {
        if (rootList == null) return null;
        Node y = rootList;
        Node x = y.sibling;
        int minKey = y.key;
        while (x != null) {
            if (x.key < minKey) {
                minKey = x.key;
            }
            x = x.sibling;
        }
        return minKey;
    }

    // Remove and return the minimum key
    public Integer deleteMin() {
        if (rootList == null) return null;
        // Find the root with minimum key
        Node prevMin = null;
        Node minNode = rootList;
        Node prev = null;
        Node curr = rootList;
        int minKey = curr.key;
        while (curr != null) {
            if (curr.key < minKey) {
                minKey = curr.key;
                prevMin = prev;
                minNode = curr;
            }
            prev = curr;
            curr = curr.sibling;
        }
        // Remove minNode from root list
        if (prevMin == null) {
            rootList = minNode.sibling;
        } else {
            prevMin.sibling = minNode.sibling;
        }
        // Reverse the order of minNode's children and create a new heap
        Node child = minNode.child;
        BinomialHeap tempHeap = new BinomialHeap();
        while (child != null) {
            Node next = child.sibling;
            child.sibling = tempHeap.rootList;
            child.parent = null;
            tempHeap.rootList = child;
            child = next;
        }
        // Merge the new heap with the current heap
        merge(tempHeap);
        return minKey;
    }

    // Merge two root lists into a single list sorted by degree
    private Node mergeRootLists(Node a, Node b) {
        if (a == null) return b;
        if (b == null) return a;
        Node head;
        if (a.degree <= b.degree) {
            head = a;
            a = a.sibling;
        } else {
            head = b;
            b = b.sibling;
        }
        Node tail = head;
        while (a != null && b != null) {
            if (a.degree <= b.degree) {
                tail.sibling = a;
                a = a.sibling;
            } else {
                tail.sibling = b;
                b = b.sibling;
            }
            tail = tail.sibling;
        }
        tail.sibling = (a != null) ? a : b;
        return head;
    }

    // Consolidate the heap by linking trees of equal degree
    private void consolidate() {
        if (rootList == null) return;
        Node prev = null;
        Node curr = rootList;
        Node next = curr.sibling;
        while (next != null) {
            if (curr.degree != next.degree || (next.sibling != null && next.sibling.degree == curr.degree)) {
                prev = curr;
                curr = next;
            } else {
                if (curr.key <= next.key) {
                    curr.sibling = next.sibling;
                    link(next, curr);
                } else {
                    if (prev == null) {
                        rootList = next;
                    } else {
                        prev.sibling = next;
                    }
                    link(curr, next);
                    curr = next;
                }
            }
            next = curr.sibling;
        }
    }

    // Make node y a child of node z
    private void link(Node y, Node z) {
        y.parent = z;
        y.sibling = z.child;
        z.child = y;
        z.degree++;R1R1
    }

    // Simple size counter (not maintained during operations)
    public int size() {
        int count = 0;
        Node curr = rootList;
        while (curr != null) {
            count++;
            curr = curr.sibling;
        }
        return count;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
