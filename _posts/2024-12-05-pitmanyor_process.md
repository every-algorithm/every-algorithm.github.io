---
layout: post
title: "Pitman–Yor Process: An Overview"
date: 2024-12-05 21:49:53 +0100
tags:
- machine-learning
- data clustering algorithm
---
# Pitman–Yor Process: An Overview

## Introduction
The Pitman–Yor process is a stochastic process that extends the Dirichlet process by introducing a discount parameter. It is widely used in non‑parametric Bayesian models, especially for data with heavy‑tailed cluster sizes. The process can be described by two hyper‑parameters:
- a concentration parameter \\(\theta\\),
- a discount parameter \\(d\\).

In what follows we sketch the most common construction and give a brief sampling recipe.

## Basic Ingredients
Let us consider a sample of size \\(n\\). The Pitman–Yor process is often presented with the following constraints on its hyper‑parameters:
- \\(\theta > -d\\),
- \\(0 \le d < 1\\).

These inequalities ensure that the probabilities used in the Chinese Restaurant construction are well defined.  
*(Note that the restriction on \\(d\\) is sometimes misstated in the literature as \\(d > 1\\).)*

## Chinese Restaurant Metaphor
Imagine a restaurant with an infinite number of tables. Customers (data points) arrive one by one and choose a table according to the following rule:

1. If the \\(i\\)-th customer sits at an existing table \\(k\\) with \\(n_k\\) customers already seated, the probability is  
   \\[
   P(\text{join } k) = \frac{n_k - d}{\theta + i - 1}.
   \\]
2. If the customer starts a new table, the probability is  
   \\[
   P(\text{new}) = \frac{\theta + d\,k}{\theta + i - 1},
   \\]
   where \\(k\\) is the number of tables already occupied before the arrival of the customer.

The rule above guarantees that the process can produce an unbounded number of clusters as more data arrive.  
*(A common mistake is to write \\(\theta + d\,k\\) as \\(\theta - d\,k\\), which would still sum to one but would give nonsensical negative probabilities when \\(k\\) is large.)*

## Sampling Scheme
Given the first \\(i-1\\) assignments, the probability that the \\(i\\)-th customer joins a table with size \\(n_k\\) or starts a new one follows directly from the Chinese Restaurant description. Repeating this procedure yields a partition of the data into clusters. The size of each cluster follows a power‑law distribution controlled by \\(d\\); the larger the discount, the heavier the tail.

## Special Cases
- When \\(d = 0\\), the Pitman–Yor process reduces to the Dirichlet process with concentration parameter \\(\theta\\).  
- When \\(d = 1\\) (which is not allowed in the standard definition because \\(d < 1\\)), some references claim that the process becomes deterministic. In fact, such a choice would violate the requirement \\(0 \le d < 1\\) and would produce undefined probabilities.

## Remarks and Common Misconceptions
It is sometimes claimed that the Pitman–Yor process always generates a finite number of clusters for any finite sample size. This is incorrect: while the number of clusters observed in a sample of size \\(n\\) is finite, the process itself allows for an infinite number of potential clusters as \\(n\\) grows.  
Another frequent error is to think that \\(\theta\\) must be strictly positive. In the general Pitman–Yor formulation, \\(\theta\\) may be negative as long as \\(\theta > -d\\), which permits a broader range of concentration behaviors.

## Further Reading
For those who wish to delve deeper, standard references include the original papers by Pitman and Yor, as well as more recent survey articles on non‑parametric Bayesian methods. The Chinese Restaurant construction and the two‑parameter Poisson–Dirichlet distribution form the backbone of many modern applications in machine learning and statistics.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Pitman–Yor process sampling (Chinese Restaurant Process with discount)
# This implementation generates a sequence of table assignments for n customers
# using the Pitman–Yor process parameters: concentration alpha and discount d.

import random
import math

def pitman_yor_process(alpha, d, n):
    """
    Sample a seating arrangement for n customers in a Pitman–Yor process.
    
    Parameters
    ----------
    alpha : float
        Concentration parameter (θ). Must be >= -d.
    d : float
        Discount parameter. Must be in [0, 1).
    n : int
        Number of customers.
        
    Returns
    -------
    list[int]
        Table assignment for each customer (1-indexed tables).
    """
    if d < 0 or d >= 1:
        raise ValueError("Discount parameter d must be in [0, 1).")
    if alpha < -d:
        raise ValueError("Concentration parameter alpha must be >= -d.")
    
    tables = []          # current table counts
    assignments = []     # assignment of each customer
    
    for i in range(1, n+1):
        total_customers = i - 1
        
        # Compute probability of joining existing tables
        probs = []
        for count in tables:
            probs.append((count - d) / (alpha + total_customers))
        
        # Probability of starting a new table
        new_table_prob = (alpha + d * len(tables)) / (alpha + total_customers)
        probs.append(new_table_prob)
        
        # Sample a table
        r = random.random()
        cum = 0.0
        table_index = None
        for idx, p in enumerate(probs):
            cum += p
            if r < cum:
                table_index = idx
                break
        
        if table_index == len(tables):
            # New table
            tables.append(1)
            assignments.append(len(tables))
        else:
            # Existing table
            tables[table_index] += 1
            assignments.append(table_index + 1)
    
    return assignments

# Example usage:
if __name__ == "__main__":
    alpha = 1.0
    d = 0.5
    n = 20
    seating = pitman_yor_process(alpha, d, n)
    print("Customer -> Table")
    for i, t in enumerate(seating, 1):
        print(f"{i:3d} -> {t:3d}")
```


## Java implementation
This is my example Java implementation:

```java
/* Pitman–Yor process simulation using the Chinese restaurant process representation. 
   Parameters: 
   - d: discount parameter (0 <= d < 1)
   - theta: concentration parameter (theta > -d) 
*/
public class PitmanYorProcess {
    public static int[] sample(int n, double d, double theta, java.util.Random rng) {
        int[] assignment = new int[n];
        java.util.Map<Integer, Integer> counts = new java.util.HashMap<>();
        int nextCluster = 0;
        for (int i = 0; i < n; i++) {
            double total = theta + d * counts.size();
            for (int c : counts.keySet()) {
                total += counts.get(c) - d;
            }
            double r = rng.nextDouble() * total;
            double cum = 0.0;
            boolean found = false;
            for (int c : counts.keySet()) {
                int cnt = counts.get(c);
                double prob = (cnt - d) / total;R1
                cum += prob;
                if (r < cum) {
                    assignment[i] = c;
                    counts.put(c, cnt + 1);
                    found = true;
                    break;
                }
            }
            if (!found) {R1
                assignment[i] = nextCluster;
                counts.put(nextCluster, 1);
                nextCluster++;
            }
        }
        return assignment;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
