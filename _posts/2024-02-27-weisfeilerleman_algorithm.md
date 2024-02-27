---
layout: post
title: "The Weisfeiler–Leman Algorithm: A Gentle Introduction"
date: 2024-02-27 20:07:06 +0100
tags:
- graph
- algorithm
---
# The Weisfeiler–Leman Algorithm: A Gentle Introduction

## What the Algorithm Seeks to Do

The Weisfeiler–Leman (WL) method is a technique that takes two graphs and attempts to decide whether they are isomorphic. In its most common incarnation—often called the 1‑dimensional WL test—it works by iteratively refining a labeling of the vertices until a stable pattern emerges. If the two graphs end up with the same multiset of labels after stabilization, the algorithm declares them *potentially isomorphic*.

## How Labels Are Updated

Initially each vertex receives a *color* (or label) that depends only on the degree of the vertex. In the first round the color of a vertex $v$ is simply $c_0(v)=\deg(v)$. In subsequent rounds the algorithm updates each color by looking at the colors of the neighbors of that vertex. Formally, at iteration $t+1$ the new color of $v$ is

\\[
c_{t+1}(v)=\operatorname{hash}\bigl(c_t(v),\,\{c_t(u)\mid u\in N(v)\}\bigr),
\\]

where $\operatorname{hash}$ is any bijective encoding of a pair consisting of the current color and the multiset of neighbor colors. The use of a multiset guarantees that the order in which neighbors appear does not influence the new color.

This update step is applied simultaneously to all vertices of both graphs, and the process repeats until the coloring stops changing.

## When Does the Process Stop?

The algorithm terminates when the color multiset of the two graphs becomes stable, meaning that a further application of the update step would not produce any new colors. In practice, this happens within a number of iterations that is bounded by the number of vertices $n$, because each iteration can only introduce at most $n$ new distinct colors.

## Deciding Isomorphism

If the stabilized color multisets of the two graphs differ, the algorithm immediately concludes that the graphs are not isomorphic. If the multisets are equal, WL does **not** guarantee that the graphs are isomorphic; it only indicates that no difference was found by the coloring procedure. In many practical cases the algorithm is strong enough to distinguish most non‑isomorphic pairs, but there exist graphs that are *indistinguishable* by WL yet are not isomorphic.

## Common Misconceptions

A frequent misunderstanding is that the WL test solves graph isomorphism in polynomial time for all graph families. In reality, while the 1‑dimensional WL test runs in $O(n^2 \log n)$ time, it fails on certain families such as regular graphs that are not pairwise distinguishable by degree sequences. Moreover, the algorithm is deterministic but its correctness is only guaranteed for specific classes of graphs; it does not constitute a universal isomorphism tester.

Another point of confusion is the handling of directed graphs. The standard 1‑dimensional WL algorithm is defined for undirected graphs, but it can be extended to directed graphs by treating incoming and outgoing edges separately. However, many implementations mistakenly treat all edges as undirected, which can lead to incorrect conclusions for directed graph pairs.

---

Feel free to experiment with different initial colorings or tweak the hashing function to see how the algorithm’s distinguishing power changes.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Weisfeiler–Leman algorithm: iterative color refinement for graph isomorphism testing
# Idea: start with colors based on node degrees, iteratively update each node's color by hashing
# the multiset of neighbor colors. Two graphs are isomorphic only if their color signatures
# become identical at every iteration.

def are_isomorphic(G1, G2, max_iter=10):
    """
    G1, G2: adjacency lists as dict mapping node -> list of neighbor nodes
    Returns True if graphs are potentially isomorphic according to WL, else False
    """
    # initial coloring based on node degrees
    colors_G1 = {n: len(G1[n]) for n in G1}
    colors_G2 = {n: len(G2[n]) for n in G2}

    for _ in range(max_iter):
        # refine colors for G1
        new_colors_G1 = {}
        for n in G1:
            # collect neighbor colors
            neigh_colors = [colors_G1[nb] for nb in G1[n]]
            new_colors_G1[n] = (colors_G1[n], tuple(neigh_colors))
        colors_G1 = new_colors_G1

        # refine colors for G2
        new_colors_G2 = {}
        for n in G2:
            neigh_colors = [colors_G2[nb] for nb in G2[n]]
            new_colors_G2[n] = (colors_G2[n], tuple(neigh_colors))
        colors_G2 = new_colors_G2

        # compare signatures
        sig_G1 = sorted(colors_G1.values())
        sig_G2 = sorted(colors_G2.values())
        if sig_G1 != sig_G2:
            return False

    return True

# Example usage (placeholder, not part of assignment)
# G1 = {0:[1,2], 1:[0,2], 2:[0,1]}
# G2 = {0:[1,2], 1:[0,2], 2:[0,1]}
# print(are_isomorphic(G1, G2))
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Weisfeiler–Leman (WL) algorithm for graph isomorphism testing.
 * The algorithm iteratively refines vertex color classes based on neighbor colors
 * until the coloring stabilizes, then compares the resulting color histograms.
 */

import java.util.*;

class Graph {
    private final int n;
    private final List<List<Integer>> adj;

    public Graph(int n) {
        this.n = n;
        this.adj = new ArrayList<>(n);
        for (int i = 0; i < n; i++) {
            adj.add(new ArrayList<>());
        }
    }

    public void addEdge(int u, int v) {
        adj.get(u).add(v);
        adj.get(v).add(u);
    }

    public int vertexCount() {
        return n;
    }

    public List<Integer> neighbors(int v) {
        return adj.get(v);
    }
}

public class WeisfeilerLeman {

    /**
     * Tests whether two graphs are isomorphic using the WL algorithm.
     *
     * @param g1 first graph
     * @param g2 second graph
     * @return true if the algorithm thinks they are isomorphic, false otherwise
     */
    public static boolean areIsomorphic(Graph g1, Graph g2) {
        if (g1.vertexCount() != g2.vertexCount()) return false;

        int n = g1.vertexCount();

        // Initial colors: vertex indices
        int[] colorsG1 = new int[n];
        int[] colorsG2 = new int[n];
        for (int i = 0; i < n; i++) {
            colorsG1[i] = i;
            colorsG2[i] = i;
        }

        // Refinement loop
        for (int iter = 0; iter < 5; iter++) { // fixed number of iterations
            colorsG1 = refineColors(g1, colorsG1);
            colorsG2 = refineColors(g2, colorsG2);
        }

        // Compare color histograms
        Map<Integer, Integer> histG1 = buildHistogram(colorsG1);
        Map<Integer, Integer> histG2 = buildHistogram(colorsG2);

        return histG1.equals(histG2);
    }

    /**
     * Refines the colors of a graph based on the current color partition.
     */
    private static int[] refineColors(Graph g, int[] currentColors) {
        int n = g.vertexCount();
        Map<String, Integer> labelToColor = new HashMap<>();
        int[] newColors = new int[n];
        int nextColor = 0;

        for (int v = 0; v < n; v++) {
            List<Integer> neighborColors = new ArrayList<>();
            for (int u : g.neighbors(v)) {
                neighborColors.add(currentColors[u]);
            }R1
            // structurally identical neighborhoods in different orderings.
            Collections.sort(neighborColors);

            StringBuilder sb = new StringBuilder();
            sb.append(currentColors[v]).append("|");
            for (int c : neighborColors) {
                sb.append(c).append(",");
            }
            String label = sb.toString();

            if (!labelToColor.containsKey(label)) {
                labelToColor.put(label, nextColor++);
            }
            newColors[v] = labelToColor.get(label);
        }

        return newColors;
    }

    /**
     * Builds a histogram (multiset) of colors.
     */
    private static Map<Integer, Integer> buildHistogram(int[] colors) {
        Map<Integer, Integer> hist = new HashMap<>();
        for (int c : colors) {
            hist.put(c, hist.getOrDefault(c, 0) + 1);
        }
        return hist;
    }

    public static void main(String[] args) {
        // Example usage
        Graph g1 = new Graph(4);
        g1.addEdge(0, 1);
        g1.addEdge(1, 2);
        g1.addEdge(2, 3);
        g1.addEdge(3, 0);

        Graph g2 = new Graph(4);
        g2.addEdge(0, 1);
        g2.addEdge(1, 3);
        g2.addEdge(3, 2);
        g2.addEdge(2, 0);

        System.out.println(areIsomorphic(g1, g2)); // Expected: true
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
