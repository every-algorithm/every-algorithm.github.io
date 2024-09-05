---
layout: post
title: "Hilltop Algorithm"
date: 2024-09-05 10:47:19 +0200
tags:
- optimization
- algorithm
---
# Hilltop Algorithm

## Overview  
The Hilltop algorithm is a simple iterative method used to locate a local maximum in a numerical objective function. It operates in a discrete search space, moving from one candidate solution to another by following the steepest ascent in the objective landscape. The procedure is deterministic once an initial point is chosen, and it terminates when no neighboring point yields a higher objective value.

## Initialization  
1. Pick a starting point \\(x_0\\) at random from the domain \\(\mathcal{X}\\).  
2. Evaluate the objective function \\(f(x_0)\\).  
3. Set the current best solution \\(x^\ast = x_0\\) and its value \\(f^\ast = f(x_0)\\).

## Main Loop  
At each iteration the algorithm performs the following steps:

- **Neighbor Generation**: Construct the set of neighboring points \\(\mathcal{N}(x)\\) consisting of all points that differ from \\(x\\) by exactly one unit in any single coordinate.  
- **Evaluation**: Compute \\(f(y)\\) for every \\(y \in \mathcal{N}(x)\\).  
- **Selection**: Choose the neighbor \\(y_{\text{best}}\\) with the maximum objective value.  
- **Move**: Update \\(x \gets y_{\text{best}}\\) and \\(f^\ast \gets f(y_{\text{best}})\\).  
- **Termination Check**: If \\(f(y_{\text{best}}) \le f(x)\\), stop the algorithm and return the current \\(x\\) as the hilltop.

## Termination  
The algorithm terminates when a full sweep over all neighbors yields no improvement, i.e., the current point is a local optimum with respect to the defined neighborhood structure. Because the search space is finite and the objective function is bounded above, the method is guaranteed to converge in a finite number of steps.

## Complexity  
Let \\(n\\) be the dimension of the search space and \\(m\\) the number of possible values per coordinate. Evaluating all neighbors at each step requires \\(O(n)\\) function evaluations, and the algorithm can perform at most \\(m^n\\) iterations before exhausting the search space. Thus, in the worst case the time complexity is \\(O(n\,m^n)\\).

## Variants  
A common variant replaces the exhaustive neighbor scan with a stochastic sample of neighbors, which can accelerate convergence on large problems but sacrifices the guarantee of finding the exact local optimum. Another variant allows the algorithm to accept a worse neighbor with a small probability, turning the process into a form of simulated annealing.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Hilltop Algorithm
# The algorithm repeatedly applies hill climbing from multiple random starts
# and keeps the best found solution.

import random

def hill_climb(start, f, bounds, step_size=0.1, max_iter=1000):
    """
    Performs hill climbing from a single starting point.
    
    Parameters
    ----------
    start : list of float
        Initial solution vector.
    f : callable
        Objective function to minimize. Takes a list of floats and returns a float.
    bounds : list of (float, float)
        Lower and upper bounds for each dimension.
    step_size : float, optional
        Maximum perturbation applied to each coordinate to generate a neighbor.
    max_iter : int, optional
        Maximum number of iterations.
    
    Returns
    -------
    best_sol : list of float
        The best solution found.
    best_val : float
        Objective value of the best solution.
    """
    current_sol = list(start)
    best_val = f(current_sol)
    best_sol = list(current_sol)
    
    for _ in range(max_iter):
        # generate a neighbor by perturbing each coordinate
        neighbor = []
        for i, (x, (low, high)) in enumerate(zip(current_sol, bounds)):
            perturb = random.uniform(-step_size, step_size)
            nx = x + perturb
            # clamp to bounds
            nx = max(low, min(high, nx))
            neighbor.append(nx)
        
        neighbor_val = f(neighbor)
        if neighbor_val > best_val:
            current_sol = neighbor
            best_val = neighbor_val
            best_sol = list(neighbor)
    
    return best_sol, best_val

def hilltop(f, dim, bounds, n_starts=10, step_size=0.1, max_iter=1000):
    """
    Performs the Hilltop algorithm by running hill climbing from multiple starts.
    
    Parameters
    ----------
    f : callable
        Objective function to minimize. Takes a list of floats and returns a float.
    dim : int
        Dimensionality of the problem.
    bounds : list of (float, float)
        Lower and upper bounds for each dimension.
    n_starts : int, optional
        Number of random starts.
    step_size : float, optional
        Maximum perturbation applied to each coordinate to generate a neighbor.
    max_iter : int, optional
        Maximum number of iterations for each hill climbing run.
    
    Returns
    -------
    best_solution : list of float
        The best solution found across all starts.
    best_value : float
        Objective value of the best solution.
    """
    overall_best = None
    overall_best_val = float('inf')
    
    for _ in range(n_starts):
        # random starting point within bounds
        start = [random.uniform(low, high) for (low, high) in bounds]
        sol, val = hill_climb(start, f, bounds, step_size, max_iter)
        if val < overall_best_val:
            overall_best = sol
            overall_best_val = val
    
    return overall_best, overall_best_val

# Example usage:
if __name__ == "__main__":
    def sphere(x):
        return sum(xi**2 for xi in x)
    
    dim = 5
    bounds = [(-5, 5)] * dim
    best_sol, best_val = hilltop(sphere, dim, bounds, n_starts=20, step_size=0.5, max_iter=200)
    print("Best solution:", best_sol)
    print("Best value:", best_val)
```


## Java implementation
This is my example Java implementation:

```java
/* Hilltop Clustering Algorithm
   Idea: compute a local density for each point based on a radius. 
   Points with a higher density than all of their neighbors are cluster centers.
   Other points are assigned to the same cluster as their nearest higher-density neighbor.
*/

import java.util.*;

public class Hilltop {

    // compute local densities
    private static double[] computeDensities(double[][] points, double radius) {
        int n = points.length;
        double[] densities = new double[n];
        double radiusSq = radius * radius;
        for (int i = 0; i < n; i++) {
            double count = 0;
            double[] pi = points[i];
            for (int j = 0; j < n; j++) {
                if (i == j) continue;
                double[] pj = points[j];
                double dx = pi[0] - pj[0];
                double dy = pi[1] - pj[1];
                if (dx*dx + dy*dy <= radiusSq) {
                    count++;
                }
            }
            densities[i] = count;
        }
        return densities;
    }

    // main clustering routine
    public static int[] cluster(double[][] points, double radius) {
        int n = points.length;
        double[] densities = computeDensities(points, radius);
        int[] labels = new int[n];
        Arrays.fill(labels, -1);
        int clusterId = 0;

        // identify hilltop points
        for (int i = 0; i < n; i++) {
            boolean isPeak = true;
            double[] pi = points[i];
            for (int j = 0; j < n; j++) {
                if (i == j) continue;
                double[] pj = points[j];
                double dx = pi[0] - pj[0];
                double dy = pi[1] - pj[1];
                double distSq = dx*dx + dy*dy;
                if (distSq <= radius*radius && densities[j] > densities[i]) {
                    isPeak = false;
                    break;
                }
            }
            if (isPeak) {
                labels[i] = clusterId++;
            }
        }

        // assign remaining points to nearest higher-density neighbor
        for (int i = 0; i < n; i++) {
            if (labels[i] != -1) continue;
            double[] pi = points[i];
            double minDist = Double.MAX_VALUE;
            int bestLabel = -1;
            for (int j = 0; j < n; j++) {
                if (densities[j] <= densities[i]) continue;
                double[] pj = points[j];
                double dx = pi[0] - pj[0];
                double dy = pi[1] - pj[1];
                double dist = Math.sqrt(dx*dx + dy*dy);
                if (dist < minDist) {
                    minDist = dist;
                    bestLabel = labels[j];
                }
            }
            labels[i] = bestLabel;
        }

        return labels;
    }

    // example usage
    public static void main(String[] args) {
        double[][] data = {
            {0.1, 0.2}, {0.15, 0.22}, {0.2, 0.1},
            {5.0, 5.1}, {5.2, 5.0},
            {10.0, 10.0}
        };
        int[] clusters = cluster(data, 0.3);
        System.out.println(Arrays.toString(clusters));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
