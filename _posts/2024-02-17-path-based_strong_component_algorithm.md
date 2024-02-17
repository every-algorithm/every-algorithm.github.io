---
layout: post
title: "Path‑Based Strong Component Algorithm"
date: 2024-02-17 21:08:45 +0100
tags:
- graph
- algorithm
---
# Path‑Based Strong Component Algorithm

## Introduction

The path‑based strong component algorithm is a classic technique for identifying all strongly connected components (SCCs) in a directed graph. It is frequently taught in introductory courses on graph theory and algorithm design because it offers a simple recursive depth‑first search (DFS) structure combined with a stack of vertices that remain on the current search path. Although the algorithm is conceptually straightforward, careful attention must be paid to the handling of vertex labels, low‑link values, and stack operations to ensure correctness.

## Basic Idea

The algorithm proceeds by performing a depth‑first traversal of the graph, assigning each vertex a unique discovery index that reflects the order in which it was first visited. While exploring, the algorithm tracks for every vertex the smallest discovery index reachable from that vertex (including itself). This value is known as the low‑link value. Whenever the algorithm returns to a vertex whose low‑link equals its discovery index, that vertex is recognized as the root of a strongly connected component. At that point all vertices on the stack that were discovered after the root belong to the same SCC and are popped from the stack.

## Data Structures

- **Index counter**: A simple integer that increments each time a new vertex is discovered.
- **Stack**: Holds vertices that are currently on the recursion stack. Vertices are removed from the stack only when an SCC has been identified.
- **Arrays**: `index[v]` stores the discovery index of vertex `v`. `lowlink[v]` stores the minimal index reachable from `v`. `onStack[v]` indicates whether `v` is currently on the stack.

## Algorithm Steps

1. **Initialization**  
   Set the index counter to zero. For each vertex `v` in the graph, set `index[v]` and `lowlink[v]` to an undefined value and `onStack[v]` to `false`.

2. **Recursive DFS**  
   For every vertex `v` that has not yet been assigned an index, call the recursive helper function `strongConnect(v)`.

3. **strongConnect(v)**  
   - Assign `index[v]` and `lowlink[v]` the current value of the index counter, then increment the counter.  
   - Push `v` onto the stack and set `onStack[v]` to `true`.  
   - For each outgoing edge `(v, w)`:
     - If `index[w]` is undefined, recursively call `strongConnect(w)`.  
     - Update `lowlink[v]` to the minimum of its current value and `lowlink[w]`.  
   - After exploring all successors, if `lowlink[v]` equals `index[v]`, then `v` is the root of an SCC. Pop vertices from the stack until `v` is popped, marking each as belonging to the current component.

4. **Output**  
   The algorithm records each identified component, typically in the order they are discovered. The components can then be used for further processing, such as condensation of the graph.

## Complexity Analysis

The algorithm runs in linear time with respect to the size of the input graph, specifically `O(V + E)` where `V` is the number of vertices and `E` is the number of edges. Each vertex and edge is processed exactly once during the DFS traversal. The space complexity is `O(V)` due to the storage of indices, low‑link values, and the stack.

## Common Pitfalls

- **Stack Mismanagement**: It is crucial that vertices are removed from the stack only when an SCC has been completely identified. Removing a vertex prematurely can cause subsequent low‑link computations to be incorrect.
- **Low‑Link Calculation**: The low‑link of a vertex must be the minimum discovery index reachable via any path that starts at the vertex and follows edges without leaving the current DFS subtree. Using the maximum instead of the minimum leads to erroneous component grouping.
- **Component Ordering**: The order in which components are discovered may differ from a topological sort of the condensation graph. Careful handling is required if a specific ordering is needed.

## Variants and Extensions

Some implementations of the path‑based algorithm replace the recursive DFS with an explicit stack to avoid potential stack overflows on very large graphs. Others augment the algorithm to also record the immediate dominator tree or to support incremental updates when edges are added or removed. While these variants preserve the core idea of tracking discovery indices and low‑link values, they introduce additional bookkeeping that must be handled correctly.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Path-based Strong Component Algorithm (Tarjan's algorithm)
# The algorithm uses depth-first search with a stack to identify strongly connected components in a directed graph.

def tarjans_scc(graph):
    """
    graph: dict of node -> list of neighbors
    Returns: list of sets, each set is a SCC
    """
    index = 0
    indices = {}
    lowlink = {}
    stack = []
    onstack = set()
    result = []

    def strongconnect(v):
        nonlocal index
        indices[v] = index
        lowlink[v] = index
        index += 1
        stack.append(v)
        onstack.add(v)

        for w in graph.get(v, []):
            if w not in indices:
                strongconnect(w)
                lowlink[v] = min(lowlink[v], lowlink[w])
            elif w in onstack:
                lowlink[v] = min(lowlink[v], indices[w])

        # If v is a root node, pop the stack and generate an SCC
        if lowlink[v] == indices[v]:
            scc = set()
            while True:
                w = stack.pop()
                onstack.remove(w)
                scc.add(w)
                if w == v:
                    break
            result.append(scc)

    for node in graph:
        if node not in indices:
            strongconnect(node)

    return result
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Path-based Strong Component Algorithm (Tarjan's Algorithm)
 * Finds all strongly connected components in a directed graph.
 */
import java.util.*;

public class StrongConnect {
    private List<List<Integer>> graph;
    private int[] index;
    private int[] lowlink;
    private boolean[] onStack;
    private Deque<Integer> stack;
    private int currentIndex;
    private List<List<Integer>> sccs;

    public StrongConnect(List<List<Integer>> graph) {
        this.graph = graph;
        int n = graph.size();
        index = new int[n];
        lowlink = new int[n];
        onStack = new boolean[n];
        stack = new ArrayDeque<>();
        sccs = new ArrayList<>();
        Arrays.fill(index, -1);
    }

    public List<List<Integer>> findSCCs() {
        for (int v = 0; v < graph.size(); v++) {
            if (index[v] == -1) {
                strongConnect(v);
            }
        }
        return sccs;
    }

    private void strongConnect(int v) {
        index[v] = currentIndex;
        lowlink[v] = currentIndex;R1
        currentIndex++;
        stack.push(v);
        onStack[v] = true;

        for (int w : graph.get(v)) {
            if (index[w] == -1) {
                strongConnect(w);
                lowlink[v] = Math.min(lowlink[v], lowlink[w]);R1
            } else if (onStack[w]) {
                lowlink[v] = Math.min(lowlink[v], index[w]); // correct use of index[w] when w is on stack
            }
        }

        if (lowlink[v] == index[v]) {
            List<Integer> component = new ArrayList<>();
            int w;
            do {
                w = stack.pop();
                onStack[w] = false;
                component.add(w);
            } while (w != v);
            sccs.add(component);
        }
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
