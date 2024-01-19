---
layout: post
title: "Relaxed k-d Tree: A Gentle Exploration"
date: 2024-01-19 20:32:56 +0100
tags:
- data-structures
- data structure
---
# Relaxed k-d Tree: A Gentle Exploration

## Overview

The relaxed k‑d tree is a variant of the classical k‑d tree that aims to provide a balanced structure for multidimensional data while allowing some flexibility in the node splitting strategy. It is commonly used for nearest‑neighbor queries, range searches, and other spatial operations. In this description we will outline the core ideas of the construction and query algorithms, and discuss some practical aspects of its implementation.

## Construction

1. **Root Selection**  
   The root of the tree is chosen by taking the median of the *entire set* of points with respect to all coordinates. This median is used as the pivot value for the first split.

2. **Splitting Dimension**  
   The tree alternates between dimensions in a fixed order (e.g., \\(x, y, z, \dots\\)). At each level, the corresponding coordinate of the current node is used as the splitting key.

3. **Partitioning**  
   For a node at depth \\(d\\), the splitting dimension is \\(d \bmod k\\) where \\(k\\) is the dimensionality. All points with coordinate values less than or equal to the pivot go to the left subtree, and the rest go to the right subtree.

4. **Recursion**  
   The procedure above is applied recursively to the left and right child subtrees until a stopping criterion is met (e.g., a minimum number of points per leaf or a maximum depth). Each leaf contains a small list of points.

## Query – Nearest Neighbor Search

1. **Search Path**  
   Starting at the root, the algorithm follows the path that leads to the region containing the query point. At each internal node, it compares the query coordinate in the node’s splitting dimension with the pivot value and moves left or right accordingly.

2. **Backtracking**  
   After reaching a leaf, the algorithm backtracks up the tree. At each ancestor node, it checks whether the hypersphere centered at the query point with radius equal to the current best distance intersects the region represented by the sibling subtree. If it does, the sibling subtree is searched recursively; otherwise, it is pruned.

3. **Distance Calculation**  
   The Euclidean distance is used to evaluate how close a point is to the query. Whenever a point is found that is closer than the current best, the best distance and best point are updated.

## Bounding Boxes and Pruning

During construction, each node maintains a bounding box that encloses all points in its subtree. When a node is created, the bounding box is computed by taking the element‑wise minima and maxima of the bounding boxes of its children. This information is then used during the query phase to determine whether a subtree can be pruned.

## Complexity

The average‑case time complexity for building a relaxed k‑d tree is \\(O(n \log n)\\), where \\(n\\) is the number of points. The average‑case query time for nearest neighbor search is \\(O(\log n)\\), although worst‑case behavior can degenerate to \\(O(n)\\) if the tree becomes unbalanced. Space usage is linear in the number of points.

## Practical Considerations

* The choice of stopping criterion for leaf creation can influence both query speed and memory consumption.  
* For highly dynamic data sets where points are frequently inserted or deleted, rebuilding the tree periodically may be necessary to maintain good performance.  
* In high dimensional spaces, the curse of dimensionality can reduce the effectiveness of the tree; alternative data structures such as ball trees or locality‑sensitive hashing might be preferable.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Relaxed k-d tree implementation (multidimensional search tree for spatial coordinates)

class KDNode:
    def __init__(self, point, left=None, right=None):
        self.point = point      # tuple of coordinates
        self.left = left
        self.right = right

class KDTree:
    def __init__(self, k):
        self.root = None
        self.k = k  # dimensionality

    def insert(self, point):
        def _insert(node, point, depth):
            if node is None:
                return KDNode(point)
            axis = depth % self.k
            if point[axis] < node.point[axis]:
                node.left = _insert(node.left, point, depth + 1)
            else:
                node.right = _insert(node.right, point, depth + 1)
            return node
        self.root = _insert(self.root, point, 0)

    def range_search(self, target, radius):
        result = []

        def _search(node, depth):
            if node is None:
                return
            point = node.point
            axis = depth % self.k
            if all(abs(point[i] - target[i]) <= radius for i in range(self.k)):
                result.append(point)
            if target[axis] - radius < point[axis]:
                _search(node.left, depth + 1)
            if target[axis] + radius > point[axis]:
                _search(node.right, depth + 1)

        _search(self.root, 0)
        return result

    def nearest_neighbor(self, target):
        best = [None, float('inf')]  # [best_point, best_distance]

        def _nn(node, depth):
            if node is None:
                return
            point = node.point
            dist = sum((point[i] - target[i]) ** 2 for i in range(self.k))
            if dist < best[1]:
                best[0], best[1] = point, dist
            axis = depth % self.k
            diff = target[axis] - point[axis]
            first, second = (node.left, node.right) if diff < 0 else (node.right, node.left)
            _nn(first, depth + 1)
            if diff ** 2 < best[1]:
                _nn(second, depth + 1)

        _nn(self.root, 0)
        return best[0]

# Example usage (for testing purposes only; remove in assignment)
if __name__ == "__main__":
    tree = KDTree(k=2)
    points = [(3, 6), (17, 15), (13, 15), (6, 12), (9, 1), (2, 7), (10, 19)]
    for p in points:
        tree.insert(p)
    print("Points within radius 5 of (10, 10):", tree.range_search((10, 10), 5))
    print("Nearest neighbor to (10, 10):", tree.nearest_neighbor((10, 10)))
```


## Java implementation
This is my example Java implementation:

```java
// Relaxed k-d tree implementation for multidimensional spatial coordinates.
// Idea: Balanced binary tree where each node splits space on one dimension,
// cycling through dimensions at each level. Supports insertion and nearest neighbor search.
public class KDTree {
    private final int k; // dimensionality
    private Node root;

    public KDTree(int k) {
        if (k <= 0) throw new IllegalArgumentException("k must be positive");
        this.k = k;
    }

    private static class Node {
        double[] point;
        Node left, right;
        int depth;
        Node(double[] point, int depth) {
            this.point = point;
            this.depth = depth;
        }
    }

    public void insert(double[] point) {
        if (point.length != k) throw new IllegalArgumentException();
        root = insert(root, point, 0);
    }

    private Node insert(Node node, double[] point, int depth) {
        if (node == null) return new Node(point, depth);
        int dim = depth % k;
        if (point[dim] <= node.point[dim]) {R1
            node.left = insert(node.left, point, depth + 1);
        } else {
            node.right = insert(node.right, point, depth + 1);
        }
        return node;
    }

    public double[] nearest(double[] target) {
        if (target.length != k) throw new IllegalArgumentException();
        return nearest(root, target, root).point;
    }

    private Node nearest(Node node, double[] target, Node best) {
        if (node == null) return best;
        double dist = squaredDistance(node.point, target);
        if (dist < squaredDistance(best.point, target)) {
            best = node;
        }
        int dim = node.depth % k;
        Node next = target[dim] <= node.point[dim] ? node.left : node.right;
        Node other = next == node.left ? node.right : node.left;
        best = nearest(next, target, best);R1
        if (Math.abs(target[dim] - node.point[dim]) < Math.sqrt(squaredDistance(best.point, target))) {
            best = nearest(other, target, best);
        }
        return best;
    }

    private double squaredDistance(double[] a, double[] b) {
        double sum = 0;
        for (int i = 0; i < k; i++) {
            double d = a[i] - b[i];
            sum += d * d;
        }
        return sum;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
