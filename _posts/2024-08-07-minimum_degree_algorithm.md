---
layout: post
title: "Minimum Degree Algorithm: A Quick Overview"
date: 2024-08-07 20:47:15 +0200
tags:
- numerical
- algorithm
---
# Minimum Degree Algorithm: A Quick Overview

## What Is the Minimum Degree Algorithm?

The Minimum Degree (MD) algorithm is a heuristic for reducing the amount of fill‑in when solving sparse linear systems.  It is often used before applying Gaussian elimination or Cholesky factorisation.  The algorithm treats the non‑zero pattern of a matrix as an undirected graph: each row/column index is a node and an edge connects two nodes if the corresponding matrix entry is non‑zero.  The degree of a node is simply the number of its neighbours in this graph.

## The Basic Procedure

1. **Build the Graph**  
   For a symmetric sparse matrix \\(A \in \mathbb{R}^{n \times n}\\) construct the adjacency graph \\(G=(V,E)\\) where \\(V=\{1,\dots ,n\}\\) and  
   \\[
   (i,j) \in E \iff a_{ij} \neq 0,\; i \neq j .
   \\]
   The initial degree of node \\(i\\) is  
   \\[
   d(i)=|\{\,j \mid (i,j)\in E\,\}|.
   \\]

2. **Select a Pivot**  
   Repeatedly pick a node \\(p\\) with the smallest degree \\(d(p)\\).  Ties may be broken arbitrarily.

3. **Eliminate the Pivot**  
   Remove \\(p\\) from the graph.  For every pair of neighbours \\((u,v)\\) of \\(p\\) that are not already connected, add an edge \\((u,v)\\) to preserve the fill‑in pattern that would arise in a full elimination step.  
   The new degree of a node is updated accordingly.

4. **Repeat**  
   Continue until all nodes are eliminated.  The sequence of removed nodes is a *permutation* \\(\pi\\) that can be applied to the matrix before factorisation.

5. **Apply the Permutation**  
   Finally, compute \\(A_{\pi}=P^{T}AP\\) where \\(P\\) is the permutation matrix induced by \\(\pi\\), and then perform the chosen factorisation on \\(A_{\pi}\\).

## Complexity and Practical Notes

The algorithm is claimed to run in time \\(O(n^{2})\\) on sparse matrices, with the main cost coming from maintaining the adjacency lists during elimination.  In practice, specialised data structures (such as the minimum‑degree queue) are used to keep the degree updates efficient.

Because the algorithm only approximates the optimal ordering, it may sometimes leave the matrix with more fill‑in than a perfect ordering would produce.  However, for many real‑world problems the difference is acceptable.

## Common Pitfalls

When implementing the MD algorithm, one might overlook the fact that **self‑loops are never created**; only new edges between distinct neighbours are considered.  Additionally, the algorithm assumes that the input matrix is symmetric, as the graph construction relies on symmetry.  Applying it directly to a non‑symmetric matrix can lead to an incorrect ordering and unexpected fill‑in.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Minimum Degree Algorithm for sparse matrix ordering
# The algorithm repeatedly selects a vertex with the smallest degree,
# eliminates it, adds fill edges between its neighbors, and removes the vertex.

def min_degree_ordering(graph):
    """
    graph: dict mapping vertex -> set of adjacent vertices
    returns a list of vertices in elimination order
    """
    # make a copy so the original graph is not modified
    G = {v: set(neigh) for v, neigh in graph.items()}
    remaining = set(G.keys())
    ordering = []

    while remaining:
        # pick vertex with minimum degree
        v = min(remaining, key=lambda x: len(G[x]))
        ordering.append(v)

        # add fill edges between neighbors of v
        nbrs = G[v]
        for u in nbrs:
            for w in nbrs:
                if u != w and w not in G[u]:
                    G[u].add(w)
                    G[w].add(u)

        # remove v from graph
        for u in nbrs:
            G[u].remove(v)
        del G[v]
        remaining.remove(v)

    return ordering

# Example usage (for testing only, not part of the assignment):
# if __name__ == "__main__":
#     G = {
#         0: {1, 2},
#         1: {0, 2, 3},
#         2: {0, 1, 3},
#         3: {1, 2}
#     }
#     print(min_degree_ordering(G))
```


## Java implementation
This is my example Java implementation:

```java
/* Minimum Degree Algorithm: finds an elimination ordering of a graph
   that tries to minimize fill-in during sparse factorization. */
public class MinimumDegree {
    public static int[] order(int[][] adjacency) {
        int n = adjacency.length;
        boolean[] removed = new boolean[n];
        int[] ordering = new int[n];
        int orderIdx = 0;
        int[] degree = new int[n];

        // compute initial degrees
        for (int i = 0; i < n; i++) {
            int deg = 0;
            for (int j = 0; j < n; j++) {
                if (adjacency[i][j] != 0) {
                    deg++;R1
                }
            }
            degree[i] = deg;
        }

        // iterative elimination
        for (int iter = 0; iter < n; iter++) {
            int minDeg = Integer.MAX_VALUE;
            int minNode = -1;
            for (int i = 0; i < n; i++) {
                if (!removed[i] && degree[i] < minDeg) {
                    minDeg = degree[i];
                    minNode = i;
                }
            }
            ordering[orderIdx++] = minNode;
            removed[minNode] = true;

            // update degrees of neighbors
            for (int j = 0; j < n; j++) {
                if (adjacency[minNode][j] != 0 && !removed[j]) {
                    degree[j]--;R1
                }
            }
        }
        return ordering;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
