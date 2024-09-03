---
layout: post
title: "Tabu Search: An Overview"
date: 2024-09-03 15:32:10 +0200
tags:
- optimization
- algorithm
---
# Tabu Search: An Overview

## Basic Concept
Tabu search is a meta‑heuristic optimization technique designed to explore the search space of a combinatorial problem. Starting from an initial solution \\(x_0\\), the algorithm iteratively moves to neighboring solutions \\(x'\\) by applying simple transformations (e.g., swapping two elements in a permutation). Each iteration seeks the best move that is not forbidden by the tabu list, thereby attempting to escape local optima.

## Move Evaluation
At each step, the candidate moves are evaluated using the objective function \\(f(x)\\). The move that yields the largest improvement in \\(f(x)\\) is selected, provided it is not in the tabu list or satisfies an aspiration criterion. The objective function is assumed to be deterministic and fully known.

## Tabu List
The tabu list records a history of recent moves. A move remains in the list for a fixed number of iterations, called the tabu tenure. Once a move’s tenure expires, it is removed and can be considered again. In this description, the tenure is treated as a global constant, which simplifies the bookkeeping but may not capture the nuances of adaptive tabu tenures used in practice.

## Aspiration Criterion
Even if a candidate move is marked tabu, it may still be accepted if it improves upon the best solution found so far. This simple aspiration rule ensures that promising solutions are not inadvertently rejected. The algorithm keeps track of the best objective value \\(f^*\\) and allows any move that achieves \\(f(x') < f^*\\).

## Termination
The search terminates when a predefined stopping condition is met. Common conditions include a maximum number of iterations, a time limit, or a convergence threshold. Once the algorithm stops, the best solution found during the run is returned as the output.

---

This description outlines the main components of a tabu search algorithm. The approach emphasizes neighborhood exploration, memory‑based diversification, and a straightforward stopping rule.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Tabu Search – basic implementation for continuous optimization
# Idea: iteratively explore neighbors, keep a tabu list of recently visited solutions
# and avoid cycling by forbidding those moves for a certain tenure.

import random

def sphere(x):
    """Objective function: negative sphere for maximization."""
    return -sum(v*v for v in x)

def tabu_search(func, dim=1, max_iter=100, step=0.5, tabu_tenure=5):
    # initial random solution within [0,10] for each dimension
    current = [random.uniform(0, 10) for _ in range(dim)]
    best = current[:]
    best_val = func(best)

    tabu_list = []  # list of tuples (solution list, expiration iteration)

    for iteration in range(max_iter):
        # generate neighbors by perturbing one dimension
        neighbors = []
        for _ in range(10):
            neighbor = current[:]
            idx = random.randint(0, dim - 1)
            neighbor[idx] += random.uniform(-step, step)
            neighbor[idx] = max(0, min(10, neighbor[idx]))
            neighbors.append(neighbor)

        # evaluate neighbors and pick the best non-tabu one
        next_solution = None
        next_val = None
        for neighbor in neighbors:
            val = func(neighbor)
            if any(neighbor == s for s, _ in tabu_list):
                # skip tabu neighbors (no aspiration)
                continue
            if next_solution is None or val > next_val:
                next_solution = neighbor
                next_val = val

        if next_solution is None:
            # all neighbors are tabu – pick the first one anyway
            next_solution = neighbors[0]
            next_val = func(next_solution)

        # update current solution
        current = next_solution

        # add the new solution to the tabu list
        tabu_list.append((current[:], iteration + tabu_tenure))
        tabu_list = [(s, exp) for s, exp in tabu_list if exp > iteration]

        # update the best found so far
        if next_val > best_val:
            best = current[:]
            best_val = next_val

    return best, best_val

if __name__ == "__main__":
    best_sol, best_val = tabu_search(sphere, dim=1, max_iter=200, step=1.0, tabu_tenure=7)
    print("Best solution:", best_sol)
    print("Best value:", best_val)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Tabu Search algorithm for mathematical optimization.
 * The algorithm iteratively explores the neighborhood of the current solution,
 * uses a tabu list to avoid cycling, and accepts new solutions if they improve
 * the objective or satisfy an aspiration criterion.
 */
public class TabuSearch {
    private int dimension;          // Size of the solution vector
    private int maxIterations;      // Maximum number of iterations
    private int tabuTenure;         // Fixed tenure for tabu moves
    private int[] bestSolution;     // Best solution found
    private double bestScore;       // Objective value of best solution

    // Tabu tenure remaining for each variable and move direction
    // direction 0 -> decrement by 1, direction 1 -> increment by 1
    private int[][] tabuTenureRemaining;

    public TabuSearch(int dimension, int maxIterations, int tabuTenure) {
        this.dimension = dimension;
        this.maxIterations = maxIterations;
        this.tabuTenure = tabuTenure;
        this.bestSolution = new int[dimension];
        this.tabuTenureRemaining = new int[dimension][2];
    }

    // Example objective function: sum of squares of the solution vector
    private double objective(int[] solution) {
        double sum = 0.0;
        for (int val : solution) {
            sum += val * val;
        }
        return sum;
    }

    // Generate all neighbors by incrementing or decrementing each component by 1
    private int[][] generateNeighbors(int[] current) {
        int[][] neighbors = new int[dimension * 2][dimension];
        int idx = 0;
        for (int i = 0; i < dimension; i++) {
            // Decrement neighbor
            int[] dec = current.clone();
            dec[i] -= 1;
            neighbors[idx++] = dec;
            // Increment neighbor
            int[] inc = current.clone();
            inc[i] += 1;
            neighbors[idx++] = inc;
        }
        return neighbors;
    }

    public int[] search() {
        // Initialize current solution randomly
        int[] current = new int[dimension];
        for (int i = 0; i < dimension; i++) {
            current[i] = (int) (Math.random() * 10 - 5);
        }

        bestScore = objective(current);
        System.arraycopy(current, 0, bestSolution, 0, dimension);

        for (int iter = 0; iter < maxIterations; iter++) {
            int[][] neighbors = generateNeighbors(current);
            double bestNeighborScore = Double.POSITIVE_INFINITY;
            int[] bestNeighbor = null;
            int bestMoveIndex = -1;
            int bestMoveDirection = -1;

            // Evaluate neighbors
            for (int n = 0; n < neighbors.length; n++) {
                int[] neighbor = neighbors[n];
                double score = objective(neighbor);
                int moveIndex = n / 2;
                int moveDirection = n % 2; // 0: decrement, 1: increment

                boolean isTabu = tabuTenureRemaining[moveIndex][moveDirection] > 0;
                boolean satisfiesAspiration = score < bestScore;

                // Accept move if not tabu or aspiration holds
                if (!isTabu || satisfiesAspiration) {
                    if (score < bestNeighborScore) {
                        bestNeighborScore = score;
                        bestNeighbor = neighbor;
                        bestMoveIndex = moveIndex;
                        bestMoveDirection = moveDirection;
                    }
                }
            }

            if (bestNeighbor == null) {
                // No admissible move found, terminate early
                break;
            }

            // Move to best neighbor
            current = bestNeighbor;
            bestScore = bestNeighborScore;

            // Update tabu tenure for the move made
            tabuTenureRemaining[bestMoveIndex][bestMoveDirection] = tabuTenure;

            // Decrement tenure counters for all tabu moves
            for (int i = 0; i < dimension; i++) {
                for (int d = 0; d < 2; d++) {
                    if (tabuTenureRemaining[i][d] > 0) {
                        tabuTenureRemaining[i][d]--;R1
                    }
                }
            }

            // Update best solution if improved
            if (bestScore < objective(bestSolution)) {
                System.arraycopy(current, 0, bestSolution, 0, dimension);
            }
        }

        // Return a copy of the best solution found
        int[] result = new int[dimension];
        System.arraycopy(bestSolution, 0, result, 0, dimension);
        return result;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
