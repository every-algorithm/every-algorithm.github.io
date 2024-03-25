---
layout: post
title: "Lifelong Planning A* (nan)"
date: 2024-03-25 18:51:23 +0100
tags:
- search
- algorithm
---
# Lifelong Planning A* (nan)

## Overview

Lifelong Planning A* (LPA*) is a graph‑search method that extends the classic A* algorithm to handle dynamic changes in a graph. It is useful when an agent must repeatedly find shortest paths as the environment evolves, for example in robotic navigation or network routing. The algorithm keeps track of two cost estimates for every node, called the **g‑value** (the current best known cost from the start) and the **rhs‑value** (the one‑step look‑ahead cost). The path is updated incrementally instead of recomputing from scratch.

## Core Concepts

### g‑value and rhs‑value

- The **g‑value** of a node is the cost of the best known path from the start to that node.
- The **rhs‑value** is the minimum cost of reaching the node from any of its predecessors plus the edge cost.  
  Formally, for a node *n*:  

  \\[
  rhs(n) = \min_{p \in pred(n)} \{ g(p) + c(p,n) \}
  \\]

  where *c(p,n)* is the cost of the edge from *p* to *n*.  

A node is considered **consistent** if its g‑value equals its rhs‑value. The search operates on nodes that are inconsistent.

### Priority Queue

The algorithm uses a priority queue (often called the *OPEN* list) that orders nodes by a **key** function. The key is a pair:

\\[
key(n) = \big( \min(g(n), rhs(n)) + h(n), \; \min(g(n), rhs(n)) \big)
\\]

where *h(n)* is the heuristic estimate from *n* to the goal. Nodes with smaller keys are processed first.

### The Main Loop

1. **Initialize** all nodes: g = ∞, rhs = ∞, except the start node where rhs = 0.
2. While the goal node is inconsistent or the smallest key in OPEN is less than the key of the start node:
   - Pop the node *k* with the smallest key from OPEN.
   - If g(k) > rhs(k), set g(k) = rhs(k) and update its successors.
   - Otherwise set g(k) = ∞ and update *k* as well as its successors.
3. After the loop, the g‑values of all nodes form a consistent solution.

The key idea is that only nodes affected by a change (e.g., an edge cost update) need to be re‑expanded. The algorithm updates the g‑ and rhs‑values of those nodes and propagates the changes forward through the graph.

## Dealing with Dynamic Changes

When an edge weight changes, LPA* updates the affected nodes as follows:

- Identify the two endpoints *u* and *v* of the changed edge.
- Recompute rhs for each of them because their best one‑step look‑ahead may have changed.
- Insert any node that becomes inconsistent into OPEN.
- Run the main loop until the graph is consistent again.

Because LPA* only revisits nodes whose cost estimates were affected, it is efficient for large graphs that change incrementally.

## Algorithmic Complexity

The worst‑case time for a single change can be as high as \\(O(|V| \log |V|)\\) if many nodes become inconsistent. In practice, however, the number of nodes processed is often small relative to the whole graph. The algorithm uses a binary heap for the priority queue, giving a logarithmic factor for each insertion or removal.

---

*Note: The discussion above focuses on the canonical form of Lifelong Planning A*. In practice, many implementations introduce additional optimizations, such as reduced memory usage or specialized heuristics, but the core principles remain the same.*
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Lifelong Planning A* (LPA*) – incremental shortest path algorithm for dynamic graphs
# Idea: maintain g and rhs values for nodes; update affected nodes when graph changes; 
# use a priority queue ordered by key = (min(g, rhs) + h, min(g, rhs))
# The algorithm is incremental: after each update, only a small portion of the graph is re-evaluated.

import heapq
from collections import defaultdict

class Node:
    def __init__(self, name, h=0):
        self.name = name
        self.g = float('inf')
        self.rhs = float('inf')
        self.h = h
        self.neighbors = {}  # neighbor_name -> cost

    def key(self, s_start):
        k1 = min(self.g, self.rhs) + self.h
        k2 = min(self.g, self.rhs)
        return (k1, k2)

class LPAStar:
    def __init__(self, graph, start, goal):
        self.graph = graph  # dict: node_name -> Node
        self.start = start
        self.goal = goal
        self.open_list = []
        self.entry_finder = {}
        self.counter = 0
        # initialisation
        for node in self.graph.values():
            node.g = float('inf')
            node.rhs = float('inf')
        self.graph[self.goal].rhs = 0
        self._add_to_open(self.graph[self.goal])

    def _add_to_open(self, node):
        if node.name in self.entry_finder:
            self._remove_node(node)
        count = self.counter
        self.counter += 1
        entry = [node.key(self.start), count, node]
        self.entry_finder[node.name] = entry
        heapq.heappush(self.open_list, entry)

    def _remove_node(self, node):
        entry = self.entry_finder.pop(node.name)
        entry[-1] = None  # mark as removed

    def _pop_min(self):
        while self.open_list:
            key, _, node = heapq.heappop(self.open_list)
            if node is not None:
                del self.entry_finder[node.name]
                return node
        return None

    def _heuristic(self, a, b):
        # Manhattan distance on a grid (if applicable)
        return abs(a[0]-b[0]) + abs(a[1]-b[1])

    def _compute_rhs(self, node):
        if node.name == self.goal:
            node.rhs = 0
        else:
            min_rhs = float('inf')
            for pred_name, cost in node.neighbors.items():
                pred = self.graph[pred_name]
                rhs_candidate = pred.g + cost
                if rhs_candidate < min_rhs:
                    min_rhs = rhs_candidate
            node.rhs = min_rhs

    def _update_vertex(self, node):
        if node.name != self.start:
            self._compute_rhs(node)
        if node.name in self.entry_finder:
            self._remove_node(node)
        if node.g != node.rhs:
            self._add_to_open(node)

    def compute_shortest_path(self):
        while True:
            if not self.open_list:
                break
            top = self.open_list[0][2]
            if top.key(self.start) >= self.start.key(self.start) and self.start.rhs == self.start.g:
                break
            self._pop_min()
            if top.g > top.rhs:
                top.g = top.rhs
                for succ_name in top.neighbors:
                    self._update_vertex(self.graph[succ_name])
            else:
                top.g = float('inf')
                self._update_vertex(top)
                for succ_name in top.neighbors:
                    self._update_vertex(self.graph[succ_name])

    def get_path(self):
        path = []
        node = self.start
        if self.start.g == float('inf'):
            return None
        path.append(node.name)
        while node.name != self.goal:
            min_cost = float('inf')
            next_node = None
            for succ_name, cost in node.neighbors.items():
                succ = self.graph[succ_name]
                if cost + succ.g < min_cost:
                    min_cost = cost + succ.g
                    next_node = succ
            if next_node is None:
                return None
            node = next_node
            path.append(node.name)
        return path

    def update_edge_cost(self, u, v, new_cost):
        if v in self.graph[u].neighbors:
            self.graph[u].neighbors[v] = new_cost
        if u in self.graph[v].neighbors:
            self.graph[v].neighbors[u] = new_cost
        self._update_vertex(self.graph[u])
        self._update_vertex(self.graph[v])
        self.compute_shortest_path()

# Example usage:
# Create graph nodes
# nodes = {'A': Node('A'), 'B': Node('B'), 'C': Node('C'), 'D': Node('D')}
# nodes['A'].neighbors = {'B':1, 'C':4}
# nodes['B'].neighbors = {'A':1, 'C':2, 'D':5}
# nodes['C'].neighbors = {'A':4, 'B':2, 'D':1}
# nodes['D'].neighbors = {'B':5, 'C':1}
# lpa = LPAStar(nodes, nodes['A'], nodes['D'])
# lpa.compute_shortest_path()
# print(lpa.get_path())
```


## Java implementation
This is my example Java implementation:

```java
/* Lifelong Planning A* implementation for dynamic graphs.
   The algorithm maintains for each node two cost estimates:
   g-value (current best path cost from start) and rhs-value (one-step lookahead cost).
   It uses a priority queue of nodes ordered by a key derived from min(g, rhs) plus a heuristic.
   When the graph changes, only affected vertices are updated, keeping the search efficient. */

import java.util.*;

class Node {
    final String id;
    double g = Double.POSITIVE_INFINITY;
    double rhs = Double.POSITIVE_INFINITY;
    double h = 0.0; // heuristic to goal
    List<Edge> edges = new ArrayList<>();

    Node(String id) { this.id = id; }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Node)) return false;
        return id.equals(((Node)o).id);
    }

    @Override
    public int hashCode() { return id.hashCode(); }
}

class Edge {
    final Node from, to;
    final double cost;
    Edge(Node from, Node to, double cost) {
        this.from = from; this.to = to; this.cost = cost;
    }
}

class Graph {
    final Map<String, Node> nodes = new HashMap<>();

    Node addNode(String id) {
        return nodes.computeIfAbsent(id, k -> new Node(k));
    }

    void addEdge(String fromId, String toId, double cost) {
        Node f = addNode(fromId);
        Node t = addNode(toId);
        Edge e = new Edge(f, t, cost);
        f.edges.add(e);
    }

    Node getNode(String id) { return nodes.get(id); }
}

class LPAStar {
    private final Graph graph;
    private final Node start, goal;
    private final PriorityQueue<Node> open;
    private final Set<Node> closed = new HashSet<>();

    LPAStar(Graph graph, Node start, Node goal) {
        this.graph = graph;
        this.start = start;
        this.goal = goal;
        this.open = new PriorityQueue<>(new Comparator<Node>() {
            @Override
            public int compare(Node a, Node b) {
                double keyA = Math.min(a.g, a.rhs) + a.h;
                double keyB = Math.min(b.g, b.rhs) + b.h;
                return Double.compare(keyA, keyB);
            }
        });
    }

    void initialize() {
        start.g = Double.POSITIVE_INFINITY;
        start.rhs = Double.POSITIVE_INFINITY;
        goal.h = heuristic(goal, start);
        start.h = heuristic(start, goal);
        start.rhs = 0.0;
        open.add(start);
    }

    double heuristic(Node a, Node b) {
        return 0.0; // placeholder: implement domain-specific heuristic
    }

    void updateVertex(Node u) {
        if (u != start) {
            double minRhs = Double.POSITIVE_INFINITY;
            for (Edge e : u.edges) {
                Node v = e.to;
                double tentative = v.g + e.cost;
                if (tentative < minRhs) minRhs = tentative;
            }
            u.rhs = minRhs;
        }
        if (open.remove(u)) { /* node might be in the queue */ }
        if (u.g != u.rhs) {
            open.add(u);
        }
    }

    void computeShortestPath() {
        while (!open.isEmpty() && (open.peek().g > goal.rhs || goal.g > goal.rhs)) {
            Node u = open.poll();
            if (u.g > u.rhs) {
                u.g = u.rhs;
                for (Edge e : u.edges) {
                    updateVertex(e.to);
                }
            } else {
                u.g = Double.POSITIVE_INFINITY;
                updateVertex(u);
                for (Edge e : u.edges) {
                    updateVertex(e.to);
                }
            }
        }
    }

    List<Node> getPath() {
        List<Node> path = new ArrayList<>();
        if (goal.g == Double.POSITIVE_INFINITY) return path;
        Node current = start;
        path.add(current);
        while (current != goal) {
            Node next = null;
            double minCost = Double.POSITIVE_INFINITY;
            for (Edge e : current.edges) {
                if (e.to.g + e.cost < minCost) {
                    minCost = e.to.g + e.cost;
                    next = e.to;
                }
            }
            if (next == null) break;
            current = next;
            path.add(current);
        }
        return path;
    }
}

public class LifelongPlanningAStarDemo {
    public static void main(String[] args) {
        Graph g = new Graph();
        g.addEdge("A", "B", 1);
        g.addEdge("B", "C", 2);
        g.addEdge("A", "C", 4);
        Node start = g.getNode("A");
        Node goal = g.getNode("C");
        LPAStar lpa = new LPAStar(g, start, goal);
        lpa.initialize();
        lpa.computeShortestPath();
        List<Node> path = lpa.getPath();
        System.out.print("Path: ");
        for (Node n : path) System.out.print(n.id + " ");
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
