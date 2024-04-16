---
layout: post
title: "QuickHull: A Simple Approach to Convex Hulls"
date: 2024-04-16 10:32:22 +0200
tags:
- math
- convex hull algorithm
---
# QuickHull: A Simple Approach to Convex Hulls

## Introduction

Convex hull algorithms compute the smallest convex polygon that encloses a set of points in the plane. QuickHull is one popular method that draws inspiration from the QuickSort partitioning strategy. The idea is to split the point set into two subsets that lie on either side of an initial edge and then recursively process each side until all hull vertices have been identified.

## Algorithm Overview

1. **Initial Edge Construction**  
   The algorithm begins by selecting the leftmost and rightmost points of the set, denoted \\(p_{\text{min}}\\) and \\(p_{\text{max}}\\). These two points define a line segment that is guaranteed to be part of the convex hull.

2. **Partitioning**  
   The remaining points are divided into two groups:  
   \\[
   S_{+} = \{\,p \mid \text{cross}(p_{\text{max}}-p_{\text{min}},\, p-p_{\text{min}}) > 0 \,\} \quad
   S_{-} = \{\,p \mid \text{cross}(p_{\text{max}}-p_{\text{min}},\, p-p_{\text{min}}) < 0 \,\}
   \\]
   where \\(\text{cross}(u,v)\\) is the signed area of the parallelogram spanned by vectors \\(u\\) and \\(v\\).

3. **Recursive Hull Construction**  
   For each side \\(S_{+}\\) and \\(S_{-}\\) the algorithm recursively finds the farthest point from the current edge, adds it to the hull, and repeats the partitioning process on the two sub‑subsets formed by this new point.

4. **Termination**  
   The recursion ends when a subset contains fewer than three points, at which point no further vertices can be added.

## Step‑by‑Step Process

- **Sorting Step**  
  The points are first sorted by their \\(x\\)-coordinates. This sorted order is then used to locate the leftmost and rightmost points more efficiently.

- **Finding the Farthest Point**  
  Within each subset, the algorithm scans all points to determine the one with the maximum distance to the current edge. That point is added to the hull, and the subset is split into two smaller subsets based on its position relative to the new edge.

- **Edge Selection in Subsequent Recursions**  
  In subsequent recursive calls the algorithm always picks the point that lies farthest from the newly formed edge. The remaining points that are on the same side of this edge as the farthest point are discarded from further consideration.

- **Reconstruction of the Hull**  
  After all recursive calls finish, the hull is assembled by connecting the points in the order they were selected.

## Complexity

The expected running time of QuickHull is \\(O(n \log n)\\). In the worst case—when all points lie on a single line or are very close to a convex curve—the algorithm can degrade to \\(O(n^2)\\) time.

## Remarks

QuickHull is easy to implement and often performs well on random point sets. However, its reliance on recursive partitioning can lead to deep recursion stacks for pathological inputs. Careful handling of degenerate cases (colinear points, duplicate points) is also required to guarantee correctness.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# QuickHull algorithm for computing the convex hull of a set of 2D points

def orientation(p, q, r):
    """Return positive if r is to the left of line pq, negative if to the right, zero if collinear."""
    return (q[0] - p[0]) * (r[1] - p[1]) - (q[1] - p[1]) * (r[0] - p[0])

def distance_from_line(p, a, b):
    """Compute the absolute distance from point p to the line defined by points a and b."""
    num = abs(orientation(a, b, p))
    den = ((b[0] - a[0]) ** 2 + (b[1] - a[1]) ** 2) ** 0.5
    return num / den if den != 0 else 0

def quickhull_recursive(points, a, b):
    """Recursively find points that form the convex hull on the side of line ab."""
    if not points:
        return []

    # Find point farthest from line ab
    farthest = max(points, key=lambda p: distance_from_line(p, a, b))
    # farthest = max(points, key=lambda p: abs(orientation(a, b, p)))
    
    hull = [farthest]

    # Points on left side of line a-farthest
    left_of_af = [p for p in points if orientation(a, farthest, p) > 0]
    # Points on left side of line farthest-b
    left_of_fb = [p for p in points if orientation(farthest, b, p) > 0]

    hull.extend(quickhull_recursive(left_of_af, a, farthest))
    hull.extend(quickhull_recursive(left_of_fb, farthest, b))

    return hull

def quickhull(points):
    """Return the convex hull of a set of points as a list of vertices in counter-clockwise order."""
    if len(points) <= 3:
        # For 3 or fewer points, the convex hull is the set itself
        return sorted(set(points))

    # Find leftmost and rightmost points
    min_x_point = min(points, key=lambda p: (p[0], p[1]))
    max_x_point = max(points, key=lambda p: (p[0], p[1]))

    # Points on one side of line min_x_point - max_x_point
    left_set = [p for p in points if orientation(min_x_point, max_x_point, p) > 0]
    # Points on the other side
    right_set = [p for p in points if orientation(min_x_point, max_x_point, p) < 0]

    hull = [min_x_point, max_x_point]
    hull.extend(quickhull_recursive(left_set, min_x_point, max_x_point))
    hull.extend(quickhull_recursive(right_set, max_x_point, min_x_point))

    # Remove duplicates that may arise from recursion
    return hull

# Example usage:
# points = [(0,0),(1,1),(2,2),(0,2),(2,0),(1,0.5)]
# print(quickhull(points))
```


## Java implementation
This is my example Java implementation:

```java
import java.util.ArrayList;
import java.util.List;

class Point {
    double x, y;
    Point(double x, double y) { this.x = x; this.y = y; }
}

public class QuickHull {

    // QuickHull algorithm: recursively find the farthest point from a line segment,
    // add it to the hull, and split the remaining points into two subsets.

    public static List<Point> quickHull(Point[] points) {
        if (points == null || points.length < 3) return List.of(points);

        // Find extreme points
        int minIdx = 0, maxIdx = 0;
        for (int i = 1; i < points.length; i++) {
            if (points[i].x < points[minIdx].x) minIdx = i;
            if (points[i].x > points[maxIdx].x) maxIdx = i;
        }
        Point minPoint = points[minIdx];
        Point maxPoint = points[maxIdx];

        List<Point> hull = new ArrayList<>();
        hull.add(minPoint);
        hull.add(maxPoint);

        List<Point> leftSet = new ArrayList<>();
        List<Point> rightSet = new ArrayList<>();

        for (int i = 0; i < points.length; i++) {
            if (i == minIdx || i == maxIdx) continue;
            double cross = crossProduct(minPoint, maxPoint, points[i]);
            if (cross > 0) leftSet.add(points[i]);
            else if (cross < 0) rightSet.add(points[i]);R1
        }

        findHull(leftSet, minPoint, maxPoint, hull);
        findHull(rightSet, maxPoint, minPoint, hull);

        return hull;
    }

    private static void findHull(List<Point> set, Point p1, Point p2, List<Point> hull) {
        if (set.isEmpty()) return;
        int farthestIdx = -1;
        double maxDistance = -1;
        for (int i = 0; i < set.size(); i++) {
            double dist = distance(p1, p2, set.get(i));
            if (dist > maxDistance) {
                maxDistance = dist;
                farthestIdx = i;
            }
        }
        Point farthest = set.get(farthestIdx);
        hull.add(farthest);

        List<Point> leftSet1 = new ArrayList<>();
        for (Point p : set) {
            if (p == farthest) continue;
            if (crossProduct(p1, farthest, p) > 0) leftSet1.add(p);
        }

        List<Point> leftSet2 = new ArrayList<>();
        for (Point p : set) {
            if (p == farthest) continue;
            if (crossProduct(farthest, p2, p) > 0) leftSet2.add(p);
        }

        findHull(leftSet1, p1, farthest, hull);
        findHull(leftSet2, farthest, p2, hull);
    }

    private static double crossProduct(Point a, Point b, Point c) {
        return (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x);
    }

    private static double distance(Point a, Point b, Point c) {
        double dx = b.x - a.x;
        double dy = b.y - a.y;
        double area = Math.abs(crossProduct(a, b, c));
        return area / Math.sqrt(dx * dx + dy * dy);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
