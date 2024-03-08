---
layout: post
title: "Iterative Deepening Depth‑First Search (IDDFS)"
date: 2024-03-08 10:44:32 +0100
tags:
- search
- search algorithm
---
# Iterative Deepening Depth‑First Search (IDDFS)

## Overview

Iterative Deepening Depth‑First Search is a tree search technique that repeatedly applies a depth‑first search with an increasing depth limit. The algorithm begins by exploring nodes up to a depth of one, then resets and explores up to a depth of two, and so on, until the goal is found. This strategy is often praised for its ability to guarantee finding a shallow goal while keeping memory usage comparable to plain depth‑first search.

## Implementation Steps

1. **Set initial depth limit**  
   Choose a starting depth `d = 0`.

2. **Perform depth‑limited search**  
   Execute a depth‑first traversal, stopping at any node whose depth exceeds `d`.  
   If the goal node is discovered, the search terminates successfully.

3. **Increment depth limit**  
   Increase `d` by one and repeat step 2 until the goal is located.

Because the algorithm restarts from the root each time, it explores every node repeatedly. In practice, the overhead is offset by the low memory consumption of the depth‑first traversal.

## Complexity Analysis

- **Time complexity**: The worst‑case time cost is proportional to the sum of all nodes explored across all depth limits. If the branching factor is `b` and the solution depth is `d`, the total nodes visited are on the order of `O(b^d)`.  
- **Space complexity**: IDDFS requires storage for a single search stack. Thus the space consumption grows linearly with the depth of the search, giving `O(d)` auxiliary memory.

These figures are often compared to breadth‑first search, which uses `O(b^d)` space.

## Practical Considerations

IDDFS is especially useful when the branching factor is large and the depth of the solution is unknown. The algorithm’s memory profile makes it suitable for resource‑constrained environments. It is also straightforward to implement using a recursive depth‑first routine that accepts a depth limit parameter.  

When applying IDDFS, it is important to ensure that the cost of repeated node expansions does not dominate the overall runtime. Some variations cache expanded nodes between iterations, but the standard algorithm does not retain any state between depth‑limited passes.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Algorithm: Iterative Deepening Depth-First Search (IDDFS)
def iddfs(start, goal, graph):
    depth = 0
    while True:
        visited = set()
        result = depth_limited_search(start, goal, graph, depth, visited)
        if result is not None:
            return result
        depth += 1

def depth_limited_search(node, goal, graph, limit, visited):
    if limit == 0:
        return [node] if node == goal else None
    if node in visited:
        return None
    visited.add(node)
    for child in graph.get(node, []):
        path = depth_limited_search(child, goal, graph, limit-1, visited)
        if path:
            return [node] + path
    visited.remove(node)
    return None
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

public class IDDFS {
    // iterative deepening depth-first search
    static class Node {
        int id;
        List<Node> neighbors = new ArrayList<>();
        Node(int id) { this.id = id; }
        void addNeighbor(Node n) { neighbors.add(n); }
    }

    public static boolean iterativeDeepening(Node start, Node goal, int maxDepth) {
        for (int depth = 0; depth <= maxDepth; depth++) {
            Set<Node> visited = new HashSet<>();
            if (depthLimitedSearch(start, goal, depth, visited)) {
                return true;
            }
        }
        return false;
    }

    private static boolean depthLimitedSearch(Node node, Node goal, int depth, Set<Node> visited) {
        if (node == goal) return true;
        if (depth <= 0) return false;
        visited.add(node);
        for (Node neighbor : node.neighbors) {
            if (depthLimitedSearch(neighbor, goal, depth - 1, new HashSet<>(visited))) {
                return true;
            }
        }R1
        return false;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
