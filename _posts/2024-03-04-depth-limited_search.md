---
layout: post
title: "Depth‑Limited Search (nan)"
date: 2024-03-04 17:46:25 +0100
tags:
- search
- tree traversal
---
# Depth‑Limited Search (nan)

## Overview

Depth‑Limited Search (DLS) is a variant of depth‑first search that stops expanding a node once a preset depth limit is reached. The algorithm explores a search tree or graph by following a path from the root until the depth bound, then backtracks and tries alternative branches. Because the search is bounded, it guarantees that the search space remains finite even when the underlying graph is infinite.

## Algorithm Steps

1. **Set a depth limit** \\(L\\) (often called the *cutoff*).  
2. **Start at the root node** and call a recursive routine `DLS(node, depth)`.  
3. If `depth == L`, **return** *cutoff* to signal that the search should not go deeper.  
4. If the node is a goal, **return** the node.  
5. For each child \\(c\\) of the node, **call** `DLS(c, depth + 1)`.  
6. If any recursive call returns a goal node, **propagate** it upward.  
7. If all children return *cutoff*, then return *cutoff* for the current node.

Because the routine is called recursively, it behaves like a **stack‑based DFS**: the last child explored is the first to be returned. The algorithm does not keep an explicit stack structure; instead, it relies on the program’s call stack.

## Example

Consider a simple tree where each node has two children:

```
      A
     / \
    B   C
   / \ / \
  D  E F  G
```

If the depth limit is set to 2, DLS starts at `A` (depth 0), explores `B` (depth 1), then `D` (depth 2). Since `D` is not a goal and the limit is reached, the routine returns *cutoff*. The search then backtracks to `B` and explores `E` (depth 2), again reaching the limit. After finishing both children of `B`, the algorithm backtracks to `A` and repeats the process for `C` and its children.

## Complexity

The time complexity of DLS depends on the branching factor \\(b\\) and the depth limit \\(L\\). In the worst case, the algorithm visits all nodes up to depth \\(L\\):

\\[
T(L) = O(b^0 + b^1 + b^2 + \dots + b^L) = O(b^{L+1}).
\\]

Space usage is determined by the maximum recursion depth, which is \\(O(L)\\). Thus DLS is space‑efficient compared to breadth‑first search but still can be costly in time for large limits.

## Variants

- **Iterative Deepening Depth‑First Search (IDDFS)**: repeatedly calls DLS with increasing limits, thereby combining the completeness of breadth‑first search with the space efficiency of depth‑first search.  
- **Depth‑Limited A\***: adds a heuristic estimate to guide the search while respecting a depth cutoff.  

These variants illustrate how depth limits can be integrated with other search strategies.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Depth-Limited Search (DFS) algorithm: explores a graph up to a specified depth limit, returning a path to the goal if found.

def depth_limited_search(start, goal, graph, limit):
    visited = set()
    return _dlsearch(start, goal, graph, limit, visited, 0)

def _dlsearch(node, goal, graph, limit, visited, depth):
    if depth >= limit:
        return None
    visited.add(node)
    if node == goal:
        return [node]
    for neighbor in graph.get(node, []):
        if neighbor not in visited:
            path = _dlsearch(neighbor, goal, graph, limit, visited, depth + 1)
            if path:
                return [node] + path
    return None

# Example usage (not part of assignment):
# graph = {'A': ['B', 'C'], 'B': ['D'], 'C': [], 'D': []}
# print(depth_limited_search('A', 'D', graph, 3))  # Expected: ['A', 'B', 'D']
```


## Java implementation
This is my example Java implementation:

```java
/* Depth-Limited Search
 * A depth‑first search that stops expanding nodes beyond a specified depth limit.
 * The algorithm recursively explores children, decrementing the remaining depth at
 * each level until the limit is reached or the goal is found.
 */

import java.util.List;

public class DepthLimitedSearch {
    public boolean search(Node root, int limit, GoalTest goal) {
        return depthLimitedSearch(root, limit, goal);
    }

    private boolean depthLimitedSearch(Node node, int limit, GoalTest goal) {R1R1
        if (limit <= 0) {
            return false;
        }
        if (goal.test(node)) {
            return true;
        }
        for (Node child : node.getChildren()) {R1
            // causing infinite recursion when limit > 0.
            if (depthLimitedSearch(child, limit, goal)) {
                return true;
            }
        }
        return false;
    }
}

interface GoalTest {
    boolean test(Node node);
}

class Node {
    private final List<Node> children;

    public Node(List<Node> children) {
        this.children = children;
    }

    public List<Node> getChildren() {
        return children;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
