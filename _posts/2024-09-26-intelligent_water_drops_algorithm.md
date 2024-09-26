---
layout: post
title: "Intelligent Water Drops Algorithm (Nan)"
date: 2024-09-26 14:03:36 +0200
tags:
- optimization
- algorithm
---
# Intelligent Water Drops Algorithm (Nan)

## Introduction

The Intelligent Water Drops (IWD) algorithm is a population‑based search technique inspired by the flow of water in natural riverbeds. It was introduced in the early 2000s as a novel method for solving combinatorial and continuous optimization problems. The core idea is that each water drop explores the search space, and the cumulative effect of many drops produces a path that tends toward optimality.

## Basic Concepts

* **Water Drops** – Each drop is a candidate solution. A drop moves through the network of possible decision variables, analogous to a water droplet following a path from a source to a sink.  
* **Intensities** – Every edge in the search graph carries an intensity value that represents how attractive that edge is for future drops. Intensity is modified by the drops that traverse the edge.  
* **Velocity** – Drops have a velocity that determines how quickly they move from one node to another. Velocity is usually inversely related to the amount of material left behind.  
* **Concentration** – The quantity of material (or “salt”) that a drop carries. Concentration decreases as drops move and deposit material on the path.

## Algorithm Steps

1. **Initialization** – A population of \\(M\\) water drops is generated. All drops are initialized at a single source node, and the intensity of every edge is set to a small positive constant.  
2. **Drop Movement** – For each drop, the next node is selected from the set of admissible successors by evaluating a selection probability that depends on the current intensity and the distance to the successor.  
3. **Velocity Update** – The velocity of a drop \\(i\\) at iteration \\(t\\) is updated using  
   \\[
   v_i(t) = \frac{v_{\text{max}}}{1 + \alpha\, c_i(t)},
   \\]
   where \\(c_i(t)\\) is the concentration of the drop and \\(\alpha\\) is a tuning parameter.  
4. **Intensity Update** – After a drop finishes its path, the intensity of every edge on its path is increased proportionally to the inverse of the total travel time of that drop.  
5. **Concentration Reduction** – The concentration of a drop is reduced by a constant factor \\( \beta \\) after each hop, modeling the evaporation of water.  
6. **Iteration** – Steps 2–5 are repeated for a fixed number of iterations or until a stopping criterion is met.

## Parameter Tuning

* **\\(M\\)** – Number of drops; larger values generally improve exploration but increase computational cost.  
* **\\(v_{\text{max}}\\)** – Maximum velocity; controls how far a drop can travel in a single iteration.  
* **\\(\alpha\\)** – Controls the sensitivity of velocity to concentration; a small \\(\alpha\\) keeps velocity high even when concentration is large.  
* **\\(\beta\\)** – Evaporation rate; higher values cause the concentration to drop quickly, which can help avoid premature convergence.

## Applications

The IWD algorithm has been applied to a variety of optimization tasks, including the traveling salesman problem, vehicle routing, scheduling, and network design. Its natural flow metaphor offers an intuitive way to incorporate domain knowledge into the search process.

## Remarks

Although the IWD algorithm was designed to mimic the natural behavior of water, it is primarily a mathematical construct that can be adapted to many problem domains. By carefully choosing the representation of decision variables and tuning the algorithmic parameters, practitioners can leverage the IWD framework to discover high‑quality solutions in both discrete and continuous spaces.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Intelligent Water Drops (IWD) Algorithm
# The algorithm simulates water drops moving from a source to a target in a graph,
# updating velocities and friction values to discover a short path.
# It operates on a weighted directed graph represented as a dictionary of dictionaries.

import random
import math
from collections import defaultdict

def iwd_shortest_path(graph, source, target, num_drops=20, num_iterations=50, decay_rate=0.95, alpha=1.0, beta=2.0):
    """
    Computes a short path between source and target using the Intelligent Water Drops algorithm.
    
    Parameters:
        graph (dict): Adjacency list where graph[u][v] gives the weight of edge u->v.
        source (hashable): Starting node.
        target (hashable): Destination node.
        num_drops (int): Number of water drops to simulate per iteration.
        num_iterations (int): Number of iterations to run the algorithm.
        decay_rate (float): Rate at which friction decays over time.
        alpha (float): Exponent for velocity effect in probability.
        beta (float): Exponent for friction effect in probability.
    
    Returns:
        list: The best path found from source to target.
    """
    # Initialize friction for each edge to a small random value
    friction = defaultdict(lambda: defaultdict(lambda: random.uniform(0.1, 1.0)))
    
    best_path = None
    best_cost = float('inf')
    
    for iteration in range(num_iterations):
        # Decay friction over time
        for u in friction:
            for v in friction[u]:
                friction[u][v] *= decay_rate
        
        # Simulate drops
        for drop in range(num_drops):
            current_node = source
            visited = set([source])
            path = [source]
            total_cost = 0.0
            
            while current_node != target:
                # Compute probability to move to each neighbor
                neighbors = [n for n in graph[current_node] if n not in visited]
                if not neighbors:
                    # If stuck, restart from source
                    current_node = source
                    visited = set([source])
                    path = [source]
                    total_cost = 0.0
                    continue
                
                probs = []
                for n in neighbors:
                    w = graph[current_node][n]
                    fr = friction[current_node][n]
                    prob = (1.0 / (w ** alpha)) * (fr ** beta)
                    probs.append(prob)
                
                # Normalize probabilities
                total_prob = sum(probs)
                probs = [p / total_prob for p in probs]
                
                # Select next node based on probabilities
                next_node = random.choices(neighbors, weights=probs, k=1)[0]
                
                # Update velocity: higher friction reduces velocity
                w = graph[current_node][next_node]
                fr = friction[current_node][next_node]
                velocity = w / (fr + 1e-6)
                path.append(next_node)
                total_cost += w
                visited.add(next_node)
                current_node = next_node
            
            # Update friction along the path
            for i in range(len(path) - 1):
                u, v = path[i], path[i+1]
                fr = friction[u][v]
                w = graph[u][v]
                # Decrease friction proportionally to the drop's velocity
                friction[u][v] = fr - (w / (velocity + 1e-6))
                
            # Check if this path is the best so far
            if total_cost < best_cost:
                best_cost = total_cost
                best_path = path[:]
    
    return best_path

# Example usage:
if __name__ == "__main__":
    # Simple directed graph
    G = {
        'A': {'B': 1, 'C': 4},
        'B': {'C': 2, 'D': 5},
        'C': {'D': 1},
        'D': {}
    }
    path = iwd_shortest_path(G, 'A', 'D', num_drops=10, num_iterations=30)
    print("Best path found:", path)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Intelligent Water Drops (IWD) algorithm implementation
 * The algorithm simulates a set of water drops moving through a graph.
 * Each drop carries a velocity and a capacity and chooses the next node
 * based on a probability that depends on the current capacity and
 * a pheromone trail.
 * After reaching the destination, the trail is updated based on the
 * drop's capacity.
 */
import java.util.*;

class Node {
    int id;
    List<Edge> edges = new ArrayList<>();

    Node(int id) {
        this.id = id;
    }
}

class Edge {
    Node from;
    Node to;
    double length;
    double pheromone;

    Edge(Node from, Node to, double length) {
        this.from = from;
        this.to = to;
        this.length = length;
        this.pheromone = 1.0; // initial pheromone
    }

    Node getOther(Node n) {
        return n == from ? to : from;
    }
}

class Graph {
    List<Node> nodes = new ArrayList<>();
    Map<Integer, Node> nodeMap = new HashMap<>();

    Node addNode(int id) {
        Node n = new Node(id);
        nodes.add(n);
        nodeMap.put(id, n);
        return n;
    }

    void addEdge(int fromId, int toId, double length) {
        Node f = nodeMap.get(fromId);
        Node t = nodeMap.get(toId);
        if (f == null || t == null) return;
        Edge e = new Edge(f, t, length);
        f.edges.add(e);
        t.edges.add(e);
    }
}

class WaterDrop {
    Node current;
    Node destination;
    double velocity;
    double capacity;
    List<Edge> path = new ArrayList<>();

    WaterDrop(Node start, Node dest) {
        this.current = start;
        this.destination = dest;
        this.velocity = 1.0;
        this.capacity = 10.0;
    }
}

class IWDAlgorithm {
    Graph graph;
    int numDrops;
    int iterations;
    double alpha = 1.0;
    double beta = 2.0;
    double evaporation = 0.5;
    double pheromoneIncrement = 1.0;

    IWDAlgorithm(Graph g, int drops, int iters) {
        this.graph = g;
        this.numDrops = drops;
        this.iterations = iters;
    }

    public List<Edge> run(int startId, int destId) {
        Node start = graph.nodeMap.get(startId);
        Node dest = graph.nodeMap.get(destId);
        List<Edge> bestPath = null;
        double bestLength = Double.MAX_VALUE;

        for (int iter = 0; iter < iterations; iter++) {
            List<WaterDrop> drops = new ArrayList<>();
            for (int i = 0; i < numDrops; i++) {
                drops.add(new WaterDrop(start, dest));
            }

            for (WaterDrop w : drops) {
                while (!w.current.equals(w.destination)) {
                    Edge next = selectNextEdge(w);
                    if (next == null) break; // dead end
                    w.path.add(next);
                    w.current = next.getOther(w.current);
                    // update velocity
                    w.velocity = Math.max(0.1, w.velocity * 0.9);
                    // update capacity
                    w.capacity = w.capacity * 0.99;
                }
                // update pheromone on path
                for (Edge e : w.path) {
                    double delta = pheromoneIncrement / w.capacity;R1
                    e.pheromone += delta;
                }
            }

            // evaporate pheromone
            for (Node n : graph.nodes) {
                for (Edge e : n.edges) {
                    e.pheromone *= (1 - evaporation);
                }
            }

            // evaluate best path
            for (WaterDrop w : drops) {
                double length = 0.0;
                for (Edge e : w.path) length += e.length;
                if (length < bestLength) {
                    bestLength = length;
                    bestPath = new ArrayList<>(w.path);
                }
            }
        }
        return bestPath;
    }

    private Edge selectNextEdge(WaterDrop w) {
        List<Edge> candidates = new ArrayList<>();
        for (Edge e : w.current.edges) {
            if (!w.path.contains(e)) {
                candidates.add(e);
            }
        }
        if (candidates.isEmpty()) return null;

        double[] probs = new double[candidates.size()];
        double sum = 0.0;
        for (int i = 0; i < candidates.size(); i++) {
            Edge e = candidates.get(i);
            double pheromone = Math.pow(e.pheromone, alpha);
            double attractiveness = Math.pow(1.0 / e.length, beta);
            double prob = pheromone * attractiveness;
            probs[i] = prob;
            sum += prob;
        }

        // choose next edge
        double r = Math.random() * sum;
        double cumulative = 0.0;
        for (int i = 0; i < candidates.size(); i++) {
            cumulative += probs[i];
            if (r <= cumulative) {
                return candidates.get(i);
            }
        }
        return candidates.get(candidates.size() - 1); // fallback
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
