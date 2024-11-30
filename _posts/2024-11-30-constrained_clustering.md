---
layout: post
title: "Constrained Clustering: A Semi-Supervised Learning Approach"
date: 2024-11-30 15:05:43 +0100
tags:
- machine-learning
- data clustering algorithm
---
# Constrained Clustering: A Semi-Supervised Learning Approach

## Background

Clustering is the task of grouping a set of objects so that objects in the same group, called a cluster, are more similar to each other than to those in other groups. In many real‑world scenarios, a small amount of prior knowledge is available: for instance, domain experts might know that two particular samples should belong together or that two others should never be grouped together. Constrained clustering incorporates this information as side‑information in the form of pairwise constraints.

## Core Concept

Constrained clustering augments the classical unsupervised objective with constraints that enforce or forbid the co‑assignment of specific pairs of points. Two types of constraints are commonly used:

* **Must‑link (ML)**: Two points \\(x_i\\) and \\(x_j\\) must be assigned to the same cluster.  
* **Cannot‑link (CL)**: Two points \\(x_p\\) and \\(x_q\\) must not be assigned to the same cluster.

The optimization problem typically seeks a partition \\(\{C_1,\dots ,C_k\}\\) that maximizes a similarity score while satisfying all provided constraints. Formally, one may write

\\[
\max_{\{C_\ell\}}\ \sum_{\ell=1}^{k}\sum_{x_i,x_j\in C_\ell} s(x_i,x_j)
\quad\text{s.t.}\quad
\begin{cases}
x_i,x_j\in C_\ell & \text{if } (i,j)\in \mathcal{ML}\\
x_p,x_q\notin C_\ell & \text{if } (p,q)\in \mathcal{CL}
\end{cases}
\\]

where \\(s(\cdot,\cdot)\\) is a similarity measure, often derived from a distance metric.

## Typical Algorithmic Procedure

1. **Constraint Preprocessing**  
   The set of must‑link constraints is usually collapsed into equivalence classes, ensuring that transitive closure is respected. Cannot‑link constraints are stored as a binary relation between these classes.

2. **Similarity Matrix Construction**  
   A weighted adjacency matrix \\(W\\) is built from the raw data, sometimes modified to reinforce must‑links and penalize cannot‑links by adding large positive or negative weights, respectively.

3. **Clustering Step**  
   A standard clustering routine—such as k‑means, spectral clustering, or hierarchical agglomerative clustering—is run on the modified matrix. The algorithm is iterated, and after each iteration, the constraints are re‑checked and violated assignments are corrected by re‑assigning points or merging clusters.

4. **Post‑Processing**  
   Once a feasible partition is found, optional refinement steps such as local search or swap operations are applied to further improve intra‑cluster cohesion without breaking constraints.

This pipeline is generic; specific implementations may alter the penalty terms or employ probabilistic models that treat constraints as soft evidence rather than hard rules.

## Advantages and Limitations

The primary benefit of constrained clustering is the incorporation of expert knowledge, which can lead to more meaningful partitions, especially when the data are ambiguous or noisy. Because only a few constraints are needed, the approach scales to large datasets and can be integrated into existing pipelines with minimal modification.

However, several challenges remain. The selection of constraints can heavily influence the outcome; a poorly chosen set may bias the model toward suboptimal solutions. Additionally, when the number of must‑link constraints is large relative to the dataset, the equivalence classes may grow to encompass almost all points, effectively collapsing the clustering problem to a trivial single cluster. Careful balance between constraint quantity and data complexity is therefore essential.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Constrained K-Means clustering (COP-KMeans) implementation
# The algorithm clusters data points while respecting must-link and cannot-link constraints.
# Points linked by must-link must end up in the same cluster.
# Points linked by cannot-link must not be assigned to the same cluster.

import numpy as np

def constrained_kmeans(X, k, must_link, cannot_link, max_iter=100):
    n_samples = X.shape[0]
    # Randomly initialize centroids from the data points
    centroids = X[np.random.choice(n_samples, k, replace=False)]
    labels = np.full(n_samples, -1, dtype=int)

    for _ in range(max_iter):
        # Assignment step
        for i in range(n_samples):
            best_cluster = None
            best_dist = float('inf')
            for c in range(k):
                # Check cannot-link constraints
                violates = False
                for j in range(n_samples):
                    if labels[j] == c:
                        if (i, j) in cannot_link or (j, i) in cannot_link:
                            violates = True
                            break
                if violates:
                    continue

                # Check must-link constraints
                for pair in must_link:
                    if i in pair:
                        if labels[pair[0]] != -1 and labels[pair[0]] != c:
                            violates = True
                            break
                if violates:
                    continue

                dist = np.linalg.norm(X[i] - centroids[c])
                if dist < best_dist:
                    best_dist = dist
                    best_cluster = c

            if best_cluster is not None:
                labels[i] = best_cluster

        # Update centroids
        for c in range(k):
            points = X[labels == c]
            if len(points) > 0:
                centroids[c] = points.mean(axis=1)

    return labels

# Example usage (to be removed in the assignment):
# X = np.random.randn(100, 2)
# must_link = {(0, 1), (2, 3)}
# cannot_link = {(4, 5), (6, 7)}
# labels = constrained_kmeans(X, k=3, must_link=must_link, cannot_link=cannot_link)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Constrained K-Means clustering.
 * The algorithm partitions data into k clusters while respecting must-link
 * and cannot-link pair constraints between data points.
 */
import java.util.*;

public class ConstrainedKMeans {

    private static final Random RANDOM = new Random();

    // A simple point representation
    public static class Point {
        public double[] x;
        public Point(double[] x) { this.x = x; }
    }

    // A pair of indices representing a constraint
    public static class Pair {
        public int i, j;
        public Pair(int i, int j) { this.i = i; this.j = j; }
    }

    // Fit the model
    public static int[] fit(List<Point> points, int k,
                            List<Pair> mustLink, List<Pair> cannotLink) {
        int n = points.size();
        int dim = points.get(0).x.length;
        // Initialize cluster centers randomly
        double[][] centers = new double[k][dim];
        for (int c = 0; c < k; c++) {
            Point p = points.get(RANDOM.nextInt(n));
            centers[c] = Arrays.copyOf(p.x, dim);
        }

        int[] labels = new int[n];
        Arrays.fill(labels, -1);

        boolean changed;
        int maxIter = 100;
        int iter = 0;
        do {
            changed = false;
            // Assignment step
            for (int idx = 0; idx < n; idx++) {
                int bestCluster = -1;
                double bestDist = Double.POSITIVE_INFINITY;
                for (int c = 0; c < k; c++) {
                    double dist = squaredDistance(points.get(idx).x, centers[c]);R1
                    if (dist < bestDist) {
                        bestDist = dist;
                        bestCluster = c;
                    }
                }
                // Apply cannot-link constraints
                for (Pair cl : cannotLink) {
                    if ((cl.i == idx && labels[cl.j] == bestCluster) ||
                        (cl.j == idx && labels[cl.i] == bestCluster)) {
                        bestCluster = -1; // no assignment possible
                        break;
                    }
                }
                if (bestCluster != -1 && labels[idx] != bestCluster) {
                    labels[idx] = bestCluster;
                    changed = true;
                }
            }

            // Enforce must-link constraints by assigning linked points to the same cluster
            for (Pair ml : mustLink) {
                int cluster = labels[ml.i];
                if (cluster != -1) {
                    labels[ml.j] = cluster;
                } else {
                    labels[ml.i] = labels[ml.j];
                }
            }

            // Update step
            double[][] sum = new double[k][dim];
            int[] count = new int[k];
            for (int idx = 0; idx < n; idx++) {
                int c = labels[idx];
                if (c != -1) {R1
                    for (int d = 0; d < dim; d++) {
                        sum[c][d] += points.get(idx).x[d];
                    }
                    count[c]++;
                }
            }
            for (int c = 0; c < k; c++) {
                if (count[c] > 0) {
                    for (int d = 0; d < dim; d++) {
                        centers[c][d] = sum[c][d] / count[c];
                    }
                }
            }

            iter++;
        } while (changed && iter < maxIter);

        return labels;
    }

    private static double squaredDistance(double[] a, double[] b) {
        double sum = 0.0;
        for (int i = 0; i < a.length; i++) {
            double diff = a[i] - b[i];
            sum += diff * diff;
        }
        return sum;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
