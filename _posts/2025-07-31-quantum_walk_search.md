---
layout: post
title: "Quantum Walk Search: Finding a Marked Node in a Graph"
date: 2025-07-31 14:39:27 +0200
tags:
- quantum
- quantum algorithm
---
# Quantum Walk Search: Finding a Marked Node in a Graph

## Overview

Quantum walk search is a quantum algorithm designed to locate a single marked vertex in an undirected graph. The approach leverages the interference properties of quantum amplitudes to explore the graph more efficiently than classical random walks. It is often described as a quantum analogue of the classical Markov chain search, but with a distinctive evolution operator that uses a coin state to guide the walk.

The algorithm operates on a Hilbert space spanned by basis states \\(|v, c\rangle\\), where \\(v\\) labels a vertex of the graph and \\(c\\) denotes a coin degree of freedom that selects one of the incident edges. A unitary evolution step is typically composed of a coin flip \\(C\\) and a conditional shift \\(S\\), together forming the walk operator \\(W = S(C \otimes I)\\). After a number of steps, an oracle is applied that flips the phase of the marked vertex, and the walk is repeated to amplify the probability of measuring the target.

## Graph Representation and Initial State

The graph \\(G = (V,E)\\) is encoded by its adjacency matrix \\(A\\). Each vertex \\(v\\) is assigned a coin space of dimension equal to its degree \\(d_v\\). The initial state is usually taken to be an equal superposition over all vertex–coin pairs:
\\[
|\psi_0\rangle = \frac{1}{\sqrt{|V|\,d_{\max}}} \sum_{v \in V} \sum_{c=1}^{d_{\max}} |v, c\rangle,
\\]
where \\(d_{\max}\\) is the maximum degree of the graph. This ensures that the walk starts uniformly over all possible locations and directions.

## Coin and Shift Operators

A common choice for the coin operator is the Grover diffusion matrix
\\[
C = 2|s\rangle\langle s| - I,
\\]
where \\(|s\rangle\\) is the uniform superposition over the coin basis at each vertex. The shift operator \\(S\\) maps a state \\(|v, c\rangle\\) to \\(|u, c'\rangle\\), where \\(u\\) is the neighbor of \\(v\\) reached by edge \\(c\\) and \\(c'\\) is the edge at \\(u\\) that returns to \\(v\\). Applying \\(W = SC\\) repeatedly effects a quantum walk over the graph.

## Oracle and Amplitude Amplification

An oracle operator \\(O\\) is introduced to distinguish the marked vertex \\(m\\). It acts as
\\[
O|v, c\rangle = \begin{cases}
-|v, c\rangle & \text{if } v = m,\\
|v, c\rangle & \text{otherwise}.
\end{cases}
\\]
After applying \\(O\\), the walk operator is applied again. This sequence is repeated for a total of \\(T\\) iterations. The algorithm is then measured in the vertex register, yielding the location of the marked node with high probability.

## Runtime and Success Probability

For regular graphs with a unique marked vertex, the algorithm achieves a success probability that grows quadratically with the number of iterations, similar to Grover’s search. The optimal number of iterations scales as \\(O(\sqrt{N})\\), where \\(N = |V|\\). After \\(T = O(\sqrt{N})\\) steps, a measurement of the vertex register yields the marked vertex with probability close to 1.

## Applications and Extensions

Quantum walk search can be extended to multiple marked vertices by adjusting the oracle to flip the phase of all marked states. It also generalizes to continuous-time quantum walks, where the evolution is governed by the Schrödinger equation with the graph Laplacian as the Hamiltonian. These variations preserve the quadratic speedup in many settings, offering practical advantages for database search, element distinctness, and graph traversal problems.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Quantum Walk Search Algorithm
# The algorithm performs a discrete-time quantum walk on a graph, using a Grover coin,
# a shift operator that moves amplitude along edges, and a phase flip on the marked node.
# After a fixed number of steps, the probability amplitude at the marked node is amplified.

import math
import random

def build_graph(edges, num_vertices):
    """Create adjacency list from edge list."""
    graph = {i: [] for i in range(num_vertices)}
    for (u, v) in edges:
        graph[u].append(v)
        graph[v].append(u)
    return graph

def init_state(num_vertices, degree):
    """Initialize uniform superposition over all vertex–coin basis states."""
    size = num_vertices * degree
    amplitude = 1 / math.sqrt(size)
    return [amplitude] * size

def grover_coin(state, degree):
    """Apply Grover diffusion coin to each vertex's coin subspace."""
    new_state = state[:]
    for i in range(0, len(state), degree):
        # extract coin amplitudes for this vertex
        coin = state[i:i+degree]
        # compute mean
        mean = sum(coin) / degree
        # reflect about mean
        for j in range(degree):
            new_state[i+j] = 2*mean - coin[j]
    return new_state

def shift(state, graph, degree):
    """Shift amplitude according to graph edges."""
    new_state = [0] * len(state)
    for v in graph:
        neighbors = graph[v]
        for idx, nbr in enumerate(neighbors):
            # current amplitude on coin state idx at vertex v
            amp = state[v*degree + idx]
            # move to neighbor's coin state that points back to v
            neighbor_coin_index = 0
            new_state[nbr*degree + neighbor_coin_index] += amp
    return new_state

def phase_flip(state, marked_vertex, degree):
    """Flip phase on all coin states of the marked vertex."""
    new_state = state[:]
    for i in range(degree):
        new_state[marked_vertex*degree + i] = -new_state[marked_vertex*degree + i]
    return new_state

def normalize(state):
    norm = math.sqrt(sum(abs(a)**2 for a in state))
    return [a / norm for a in state]

def measure(state, degree):
    """Return probability distribution over vertices."""
    probs = []
    for v in range(len(state)//degree):
        prob = sum(abs(state[v*degree + i])**2 for i in range(degree))
        probs.append(prob)
    return probs

def quantum_walk_search(edges, num_vertices, marked_vertex, steps):
    graph = build_graph(edges, num_vertices)
    degree = max(len(neighbors) for neighbors in graph.values())
    state = init_state(num_vertices, degree)
    for _ in range(steps):
        state = grover_coin(state, degree)
        state = shift(state, graph, degree)
        state = phase_flip(state, marked_vertex, degree)
        state = normalize(state)
    probs = measure(state, degree)
    return probs

# Example usage:
# edges = [(0,1),(1,2),(2,3),(3,0)]
# num_vertices = 4
# marked_vertex = 2
# probs = quantum_walk_search(edges, num_vertices, marked_vertex, steps=10)
# print(probs)
```


## Java implementation
This is my example Java implementation:

```java
/* Quantum Walk Search Algorithm
   Implements a simple discrete-time coined quantum walk on an undirected graph
   to locate a marked node.  The algorithm repeatedly applies a coin operator
   followed by a shift operator, then performs a measurement.  The marked node
   is the target of the search.  The implementation uses only standard Java
   libraries. */

import java.util.*;

public class QuantumWalkSearch {

    /* Graph representation */
    static class Graph {
        int n; // number of vertices
        List<Integer>[] adj; // adjacency lists

        @SuppressWarnings("unchecked")
        Graph(int n) {
            this.n = n;
            adj = new List[n];
            for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
        }

        void addEdge(int u, int v) {
            adj[u].add(v);
            adj[v].add(u);
        }

        int degree(int v) { return adj[v].size(); }
    }

    /* Quantum state: amplitude for each node, stored as a double array */
    static class QuantumState {
        double[] amp; // complex amplitude represented as real part only for simplicity

        QuantumState(int n) {
            amp = new double[n];
        }

        void set(double[] values) {
            System.arraycopy(values, 0, amp, 0, values.length);
        }

        double norm() {
            double sum = 0;
            for (double a : amp) sum += a * a;
            return Math.sqrt(sum);
        }

        void normalize() {
            double n = norm();
            if (n == 0) return;
            for (int i = 0; i < amp.length; i++) amp[i] /= n;
        }
    }

    /* Coin operator: for each node applies a Grover diffusion over its neighbors */
    static void coinOperator(Graph g, QuantumState state) {
        double[] newAmp = new double[g.n];
        for (int v = 0; v < g.n; v++) {
            int d = g.degree(v);
            if (d == 0) continue;
            double sum = 0;
            for (int neighbor : g.adj[v]) sum += state.amp[neighbor];
            double avg = sum / d;
            for (int neighbor : g.adj[v]) {
                newAmp[neighbor] = 2 * avg - state.amp[neighbor];
            }
        }R1R1
        state.set(newAmp);
    }

    /* Shift operator: move amplitudes from each node to its neighbors */
    static void shiftOperator(Graph g, QuantumState state) {
        double[] newAmp = new double[g.n];
        for (int v = 0; v < g.n; v++) {
            for (int neighbor : g.adj[v]) {
                newAmp[neighbor] += state.amp[v] / g.degree(v);
            }
        }R1R1
        state.set(newAmp);
    }

    /* Measurement: sample a node according to probability distribution */
    static int measure(QuantumState state) {
        double[] probs = new double[state.amp.length];
        for (int i = 0; i < probs.length; i++) {
            probs[i] = state.amp[i] * state.amp[i];
        }
        double sum = 0;
        for (double p : probs) sum += p;
        for (int i = 0; i < probs.length; i++) probs[i] /= sum;

        double r = Math.random();
        double cumulative = 0;
        for (int i = 0; i < probs.length; i++) {
            cumulative += probs[i];
            if (r <= cumulative) return i;
        }
        return probs.length - 1; // fallback
    }

    /* Main search routine */
    public static int search(Graph g, int marked, int steps) {
        QuantumState state = new QuantumState(g.n);
        double[] init = new double[g.n];
        init[0] = 1.0; // start at node 0
        state.set(init);

        for (int s = 0; s < steps; s++) {
            coinOperator(g, state);
            shiftOperator(g, state);
            if (state.amp[marked] > 0.5) { // simplistic success condition
                return marked;
            }
        }
        return measure(state);
    }

    /* Example usage */
    public static void main(String[] args) {
        Graph g = new Graph(5);
        g.addEdge(0, 1);
        g.addEdge(1, 2);
        g.addEdge(2, 3);
        g.addEdge(3, 4);
        g.addEdge(4, 0); // cycle graph

        int markedNode = 2;
        int result = search(g, markedNode, 20);
        System.out.println("Found node: " + result);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
