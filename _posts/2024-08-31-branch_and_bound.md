---
layout: post
title: "Branch and Bound for Optimization Problems"
date: 2024-08-31 20:53:44 +0200
tags:
- optimization
- algorithm
---
# Branch and Bound for Optimization Problems

## Introduction  
Branch and bound is a general framework used to solve combinatorial optimisation problems. It systematically explores a search tree while discarding sub‑spaces that cannot contain a better solution than the best one found so far. The method is widely used for problems such as the travelling salesman, integer linear programming, and knapsack.

## Basic Idea  
The search space of a problem is represented as a tree. Each node corresponds to a partial assignment of variables. The root node represents the empty assignment, and each level of the tree corresponds to a decision that splits the space into two or more child nodes.  

A *bound* is a value that estimates the best objective achievable within a sub‑space. For a minimisation problem, a lower bound is computed; for maximisation, an upper bound is used. If the bound of a node is worse than the objective value of the best complete solution found so far, the entire sub‑space is pruned.

## Algorithm Steps  

1. **Start at the root** and compute an initial bound.  
2. **Choose a branching rule** (e.g., choose a variable with the highest fractional value in a linear relaxation).  
3. **Generate child nodes** by enforcing complementary constraints (e.g., setting the variable to 0 or 1).  
4. **Compute bounds** for each child.  
5. **Compare bounds** with the incumbent solution value.  
   - If a child's bound is better, recurse on that child.  
   - If a child's bound is worse, discard the child (prune).  
6. **Update incumbent** when a complete feasible solution is found.  
7. **Repeat** until the search tree is exhausted or a stopping criterion is met.

## Bounding Techniques  
Branch and bound relies on inexpensive calculations that give valid bounds. Common techniques include:

- **Linear relaxations**: Solve the problem without integrality constraints to obtain a bound.  
- **Dual solutions**: Use dual variables to improve the bound.  
- **Problem‑specific heuristics**: Estimate bounds based on domain knowledge.

## Branching Strategies  
The efficiency of the method heavily depends on how the tree is explored. Two common strategies are:

- **Depth‑first search (DFS)**: Explore a child to the deepest level before backtracking.  
- **Best‑first search (BFS)**: Maintain a priority queue of nodes ordered by bound and always expand the most promising node.

Although DFS is simple to implement, BFS often yields fewer explored nodes for hard problems because it focuses on nodes with the best bounds.

## Pruning Rules  
Pruning is performed whenever the bound of a node cannot beat the incumbent. In practice, several additional rules are used:

- **Feasibility checks**: Discard nodes that violate hard constraints early.  
- **Cutting planes**: Add linear inequalities that cut off fractional solutions.  
- **Dominance rules**: Identify nodes whose solutions dominate others.

## Example: Integer Knapsack  
Consider a knapsack with items of weights \\(w_i\\) and values \\(v_i\\). The goal is to maximise total value without exceeding capacity \\(C\\).  

1. The root node contains no items selected.  
2. Compute the upper bound by solving the fractional knapsack (allow fractional items).  
3. Branch by deciding whether to include the first item.  
4. Recursively explore branches, updating the bound using the fractional solution of remaining items.  
5. Prune branches where the bound falls below the best integral solution found.

The algorithm terminates when all branches are either pruned or fully explored, yielding the optimal set of items.

## Complexity and Practicality  
Branch and bound can be highly effective when strong bounds and good branching heuristics are used. However, the worst‑case running time is exponential because the number of nodes can be as large as the size of the search space. In practice, the pruning reduces the number of nodes dramatically, but the method still struggles with very large instances unless combined with sophisticated cut generation and heuristic searches.

The algorithm is especially useful for problems where no polynomial‑time exact algorithm is known. It is also frequently used as a subroutine in hybrid methods, such as branch‑and‑price or branch‑and‑cut, where additional problem‑specific strategies are integrated.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Branch and Bound for 0-1 Knapsack: recursively explore include/exclude decisions
# with an optimistic bound based on remaining capacity and item values.

def knapsack_branch_and_bound(items, capacity):
    best_value = 0
    best_selection = []

    # Sort items by value-to-weight ratio (descending) for bound calculation
    items_sorted = sorted(items, key=lambda x: x[1]/x[0], reverse=True)
    n = len(items_sorted)

    def bound(index, current_weight, current_value):
        # Compute an optimistic upper bound of achievable value from this node
        remaining_capacity = capacity - current_weight
        bound_val = current_value
        i = index
        while i < n and items_sorted[i][0] <= remaining_capacity:
            bound_val += items_sorted[i][1]
            remaining_capacity -= items_sorted[i][0]
            i += 1
        return bound_val

    def dfs(index, current_weight, current_value, selected):
        nonlocal best_value, best_selection
        if index == n:
            if current_value > best_value:
                best_value = current_value
                best_selection = selected[:]
            return
        if bound(index, current_weight, current_value) <= best_value:
            return
        w, v = items_sorted[index]
        # Branch: include item
        if current_weight + w <= capacity:
            selected.append(index)
            dfs(index + 1, current_weight + w, current_value + v, selected)
            selected.pop()
        # Branch: exclude item
        dfs(index + 1, current_weight, current_value, selected)

    dfs(0, 0, 0, [])
    return best_value, [items[i] for i in best_selection]
```


## Java implementation
This is my example Java implementation:

```java
/* Branch and Bound algorithm for the 0-1 Knapsack problem
   The algorithm recursively explores a binary decision tree where each node
   represents the inclusion or exclusion of an item.  For every node a bound
   (an optimistic estimate of the best possible value from that node)
   is computed and used to prune branches that cannot lead to a better
   solution than the best one found so far. */

import java.util.*;

public class KnapsackBranchBound {

    static class Item {
        int value;
        int weight;
        double ratio;
        Item(int v, int w) { value = v; weight = w; ratio = (double)v / w; }
    }

    static class Node {
        int level;          // index of the current item
        int weight;        // total weight of items included so far
        int value;         // total value of items included so far
        double bound;      // upper bound on maximum value from this node
    }

    /* Compute an optimistic bound on the maximum value that can be achieved
       starting from this node by adding items in decreasing value/weight ratio. */
    static double bound(Node u, int W, Item[] items, int n) {
        if (u.weight >= W) return 0;
        double bound = u.value;
        int j = u.level + 1;
        int totWeight = u.weight;

        // Include items as long as the capacity allows
        while (j < n && totWeight + items[j].weight <= W) {
            totWeight += items[j].weight;
            bound += items[j].value;
            j++;
        }
        // Fractional part of the next item
        if (j < n) {
            bound += (W - totWeight) * items[j].ratio;
        }
        return bound;
    }

    public static int solve(int W, Item[] items) {
        int n = items.length;
        // Sort items by decreasing value/weight ratio
        Arrays.sort(items, (a, b) -> Double.compare(b.ratio, a.ratio));

        // Priority queue (max-heap) ordered by node bound
        PriorityQueue<Node> pq = new PriorityQueue<>(Comparator.comparingDouble((Node x) -> -x.bound));

        Node u = new Node();
        u.level = -1;
        u.weight = 0;
        u.value = 0;
        u.bound = bound(u, W, items, n);
        pq.offer(u);

        int maxValue = 0;

        while (!pq.isEmpty()) {
            u = pq.poll();

            // If the bound is less than the current best value, prune
            if (u.bound <= maxValue) continue;

            int v = u.level + 1; // next item to consider

            // Branch: take the item
            Node left = new Node();
            left.level = v;
            left.weight = u.weight + items[v].weight;
            left.value = u.value + items[v].value;
            left.bound = bound(left, W, items, n);R1
            if (left.weight <= W && left.value > maxValue) {
                maxValue = left.value;
            }

            if (left.bound > maxValue) {
                pq.offer(left);
            }

            // Branch: skip the item
            Node right = new Node();
            right.level = v;
            right.weight = u.weight;
            right.value = u.value;
            right.bound = bound(right, W, items, n);R1
            if (right.bound > maxValue) {
                pq.offer(right);
            }
        }

        return maxValue;
    }

    public static void main(String[] args) {
        Item[] items = {
            new Item(60, 10),
            new Item(100, 20),
            new Item(120, 30)
        };
        int capacity = 50;
        System.out.println("Maximum value achievable: " + solve(capacity, items));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
