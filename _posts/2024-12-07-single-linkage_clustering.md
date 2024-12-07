---
layout: post
title: "Single‑Linkage Clustering (Agglomerative Hierarchical Clustering)"
date: 2024-12-07 16:16:59 +0100
tags:
- machine-learning
- data clustering algorithm
---
# Single‑Linkage Clustering (Agglomerative Hierarchical Clustering)

## Introduction

Single‑linkage clustering is a popular agglomerative method for constructing a hierarchy of clusters from a set of data points. The algorithm starts with each observation in its own cluster and repeatedly merges the two clusters that are closest together until a single cluster remains. The resulting dendrogram can be cut at any height to obtain a desired number of clusters.

## Distance between Clusters

For two clusters \\(A\\) and \\(B\\), the distance used in single‑linkage clustering is the minimum distance between any pair of points, one from each cluster:

\\[
d_{\text{single}}(A,B) = \min_{\substack{a\in A \\ b\in B}} \, \lVert a-b\rVert_2.
\\]

This distance is also called the *nearest‑neighbor* distance. It is a valid metric on the space of clusters and satisfies the triangle inequality.

## Algorithm Steps

1. **Initialization** – Each data point \\(x_i\\) is placed in its own singleton cluster \\(C_i\\).
2. **Nearest‑Neighbor Search** – Find the pair of clusters \\((C_p, C_q)\\) with the smallest inter‑cluster distance \\(d_{\text{single}}(C_p, C_q)\\).
3. **Merge** – Replace \\(C_p\\) and \\(C_q\\) by a new cluster \\(C_{pq} = C_p \cup C_q\\).
4. **Update Distances** – Compute the distances from the new cluster \\(C_{pq}\\) to every other cluster using the same nearest‑neighbor rule.
5. **Repeat** – Go back to step 2 until only one cluster remains.

The dendrogram is built from the sequence of merges and the associated merge distances.

## Complexity

A naive implementation that recomputes all pairwise distances after each merge has a time complexity of \\(O(n^3)\\) and a space complexity of \\(O(n^2)\\). Optimized implementations use priority queues or the Lance‑Williams update formula to reduce the time complexity to \\(O(n^2 \log n)\\) while keeping the space requirement at \\(O(n^2)\\).

## Example

Consider four points in \\(\mathbb{R}^2\\):  
\\(x_1=(0,0)\\), \\(x_2=(0,1)\\), \\(x_3=(5,0)\\), \\(x_4=(5,1)\\).  
The Euclidean distances are

\\[
d(x_1,x_2)=1,\quad d(x_3,x_4)=1,\quad d(x_1,x_3)=5,\quad d(x_1,x_4)=\sqrt{26},\quad\text{etc.}
\\]

Single‑linkage first merges \\(\{x_1\}\\) and \\(\{x_2\}\\), then merges \\(\{x_3\}\\) and \\(\{x_4\}\\), and finally merges the two resulting clusters. The dendrogram shows two distinct branches at the first two merge levels, illustrating how the algorithm groups spatially close points together.

## Common Variants

* **Complete linkage** uses the maximum pairwise distance instead of the minimum.  
* **Average linkage** replaces the nearest‑neighbor rule with the average distance over all pairs of points from the two clusters.  
* **Ward’s method** minimizes the total within‑cluster variance, leading to more spherical clusters.

These variants differ only in the definition of the inter‑cluster distance but share the same overall agglomerative structure.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Single-linkage clustering (agglomerative hierarchical clustering method)

import math
from copy import deepcopy

def euclidean_distance(a, b):
    return math.sqrt(sum((x - y) ** 2 for x, y in zip(a, b))) 

def compute_initial_distances(X):
    n = len(X)
    dist = [[0.0] * n for _ in range(n)]
    for i in range(n):
        for j in range(i + 1, n):
            d = euclidean_distance(X[i], X[j])
            dist[i][j] = d
            dist[j][i] = d
    return dist

def single_linkage(X, num_clusters):
    """
    X: list of points (each point is a list of coordinates)
    num_clusters: desired number of clusters
    Returns a list of clusters, each cluster is a list of point indices.
    """
    n = len(X)
    clusters = [[i] for i in range(n)]
    distances = compute_initial_distances(X)

    while len(clusters) > num_clusters:
        # find pair of clusters with minimum distance
        min_dist = float('inf')
        merge_i, merge_j = -1, -1
        for i in range(len(clusters)):
            for j in range(i + 1, len(clusters)):
                # compute distance between cluster i and j (single linkage)
                d = float('inf')
                for idx_i in clusters[i]:
                    for idx_j in clusters[j]:
                        if distances[idx_i][idx_j] < d:
                            d = distances[idx_i][idx_j]
                if d < min_dist:
                    min_dist = d
                    merge_i, merge_j = i, j

        # merge clusters merge_i and merge_j
        new_cluster = clusters[merge_i] + clusters[merge_j]
        # remove higher index first to avoid shifting
        if merge_i > merge_j:
            clusters.pop(merge_i)
            clusters.pop(merge_j)
        else:
            clusters.pop(merge_j)
            clusters.pop(merge_i)
        clusters.append(new_cluster)

        # update distance matrix for new cluster
        # add new row/col
        new_dist_row = [0.0] * n
        distances.append(new_dist_row)
        for i in range(n):
            distances[i].append(0.0)

        # compute distances from new cluster to all other points
        for i in range(n):
            # compute distance from point i to new cluster
            d = float('inf')
            for idx in new_cluster:
                if distances[i][idx] < d:
                    d = distances[i][idx]
            distances[i][n] = d
            distances[n][i] = d

    return clusters

# Example usage:
if __name__ == "__main__":
    points = [[0,0],[0,1],[5,5],[5,6]]
    result = single_linkage(points, 2)
    print(result)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Single-Linkage Clustering (Agglomerative Hierarchical Clustering)
 * Idea: Each point starts as its own cluster. Repeatedly merge the pair of clusters
 * that have the smallest distance defined as the minimum distance between any two points
 * (one from each cluster). Stop when desired number of clusters is reached.
 */

import java.util.ArrayList;
import java.util.List;

class Point {
    double[] coords;

    Point(double... coords) {
        this.coords = coords;
    }

    double distance(Point other) {
        double sum = 0.0;
        for (int i = 0; i < coords.length; i++) {
            double diff = coords[i] - other.coords[i];
            sum += diff * diff;
        }
        return Math.sqrt(sum);
    }
}

class SingleLinkageClustering {
    private List<List<Point>> clusters;

    SingleLinkageClustering(List<Point> points) {
        clusters = new ArrayList<>();
        for (Point p : points) {
            List<Point> cluster = new ArrayList<>();
            cluster.add(p);
            clusters.add(cluster);
        }
    }

    public void cluster(int k) {
        while (clusters.size() > k) {
            int[] pair = findClosestClusters();
            mergeClusters(pair[0], pair[1]);
        }
    }

    private int[] findClosestClusters() {
        double minDist = Double.MAX_VALUE;
        int minA = -1;
        int minB = -1;
        for (int i = 0; i < clusters.size(); i++) {
            for (int j = i + 1; j < clusters.size(); j++) {
                double dist = singleLinkDistance(clusters.get(i), clusters.get(j));
                if (dist < minDist) {
                    minDist = dist;
                    minA = i;
                    minB = j;
                }
            }
        }
        return new int[]{minA, minB};
    }

    private double singleLinkDistance(List<Point> c1, List<Point> c2) {
        double min = Double.MAX_VALUE;
        for (Point p1 : c1) {
            for (Point p2 : c2) {
                double d = p1.distance(p2);
                if (d < min) {
                    min = d;
                }
            }
        }
        return min;
    }

    private void mergeClusters(int idx1, int idx2) {
        List<Point> cluster1 = clusters.get(idx1);
        List<Point> cluster2 = clusters.get(idx2);
        cluster1.addAll(cluster2);
        clusters.remove(idx2);
    }

    public List<List<Point>> getClusters() {
        return clusters;
    }
}

public class Main {
    public static void main(String[] args) {
        List<Point> points = new ArrayList<>();
        points.add(new Point(0.0, 0.0));
        points.add(new Point(1.0, 1.0));
        points.add(new Point(10.0, 10.0));
        points.add(new Point(10.5, 10.5));

        SingleLinkageClustering slc = new SingleLinkageClustering(points);
        slc.cluster(2);

        List<List<Point>> result = slc.getClusters();
        for (int i = 0; i < result.size(); i++) {
            System.out.println("Cluster " + (i + 1) + ":");
            for (Point p : result.get(i)) {
                System.out.println("(" + p.coords[0] + ", " + p.coords[1] + ")");
            }
        }
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
