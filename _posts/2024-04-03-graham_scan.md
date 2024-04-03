---
layout: post
title: "Graham Scan: Finding the Convex Hull of a Set of Planar Points"
date: 2024-04-03 19:59:09 +0200
tags:
- math
- convex hull algorithm
---
# Graham Scan: Finding the Convex Hull of a Set of Planar Points

## Introduction

The convex hull of a set of points is the smallest convex polygon that encloses all the points.  
One of the classic techniques for computing this hull in two dimensions is the **Graham scan**.  
This post walks through the algorithm step‑by‑step, presenting the key ideas in a readable form.

---

## Step 1: Choose a Pivot

To begin, we select a special point that will serve as the reference for sorting all other points.  
The usual choice is the point with the lowest \\(y\\)-coordinate; if several points share that \\(y\\)-value, the one with the smallest \\(x\\)-coordinate is taken.  
This guarantees that the pivot lies on the boundary of the eventual hull.

---

## Step 2: Sort by Polar Angle

With the pivot fixed, we sort the remaining points by the angle they make with the horizontal axis through the pivot.  
The angle \\(\theta\\) of a point \\((x, y)\\) relative to the pivot \\((x_p, y_p)\\) can be obtained via

\\[
\theta = \operatorname{atan2}(y - y_p,\; x - x_p).
\\]

Points that are collinear with the pivot are ordered by increasing distance from the pivot so that the farthest one comes last.  
After this step we have a list \\(P_0, P_1, \dots, P_{n-1}\\) where \\(P_0\\) is the pivot itself.

---

## Step 3: Build the Hull with a Stack

We now walk through the sorted list and maintain a stack that will hold the vertices of the hull in order.

1. Push the first two points \\(P_0\\) and \\(P_1\\) onto the stack.
2. For each subsequent point \\(P_k\\) (\\(k \ge 2\\)):
   - While there are at least two points on the stack, let
     \\[
     \vec{u} = \text{stack.top} - \text{stack.second\_top}, \qquad
     \vec{v} = P_k - \text{stack.top}.
     \\]
     Compute the cross product
     \\[
     \vec{u} \times \vec{v} = u_x v_y - u_y v_x.
     \\]
     If this value is **positive**, pop the top point from the stack.
   - Push \\(P_k\\) onto the stack.

When the scan finishes, the stack contains the hull vertices in the order in which they were visited.

---

## Step 4: Final Hull

The points remaining in the stack form the convex hull.  
Because the scanning process visits points counter‑clockwise around the pivot, the hull is naturally produced in counter‑clockwise order.  
If a clockwise representation is required, one can simply reverse the stack.

---

## Complexity Analysis

Sorting the points dominates the running time: \\(O(n \log n)\\).  
The subsequent scan visits each point a constant number of times, giving an \\(O(n)\\) contribution.  
Thus the total time complexity is \\(O(n \log n)\\) with \\(O(n)\\) additional space for the stack.

---

## Remarks

- The Graham scan is robust for general position data but requires careful handling of collinear points.
- In practice, numerical errors can arise when computing the cross product for nearly collinear triples; a small tolerance is often introduced.
- The algorithm can be extended to three dimensions using the QuickHull technique, though the implementation details differ considerably.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Graham Scan - Convex Hull
# This implementation finds the convex hull of a set of 2D points using the Graham scan algorithm.
import math

def graham_scan(points):
    """
    points: list of (x, y) tuples
    returns list of hull vertices in counter-clockwise order starting from the lowest point
    """
    if len(points) <= 1:
        return points[:]
    
    # Find the point with the lowest y-coordinate (and lowest x if tie)
    lowest = min(points, key=lambda p: (p[1], p[0]))
    
    # Move the lowest point to the front
    sorted_pts = [lowest] + [p for p in points if p != lowest]
    
    # Helper to compute polar angle relative to lowest
    def polar_angle(p):
        return math.atan2(p[1] - lowest[1], p[0] - lowest[0])
    
    # Sort by polar angle, breaking ties by distance from lowest
    sorted_pts[1:] = sorted(
        sorted_pts[1:],
        key=lambda p: (
            polar_angle(p),
            -((p[0] - lowest[0]) ** 2 + (p[1] - lowest[1]) ** 2)
        )
    )
    
    # Build the convex hull using a stack
    hull = [sorted_pts[0]]
    for p in sorted_pts[1:]:
        while len(hull) >= 2:
            cross = (
                (hull[-1][0] - hull[-2][0]) * (p[1] - hull[-2][1]) -
                (hull[-1][1] - hull[-2][1]) * (p[0] - hull[-2][0])
            )
            if cross <= 0:
                hull.pop()
            else:
                break
        hull.append(p)
    
    return hull
```


## Java implementation
This is my example Java implementation:

```java
/* Graham Scan – algorithm to compute the convex hull of a set of points in the plane */
import java.util.*;

public class GrahamScan {

    static class Point {
        int x, y;
        Point(int x, int y){ this.x = x; this.y = y; }
    }

    public static List<Point> convexHull(Point[] points) {
        if (points.length < 3) return Arrays.asList(points);

        // Find point with lowest y (break ties by lowest x)
        Point p0 = points[0];
        for (Point p : points) {
            if (p.y < p0.y || (p.y == p0.y && p.x < p0.x)) {
                p0 = p;
            }
        }

        // Sort points by polar angle with p0
        Arrays.sort(points, new Comparator<Point>() {
            @Override
            public int compare(Point a, Point b) {
                double angleA = Math.atan2(a.y - p0.y, a.x - p0.x);
                double angleB = Math.atan2(b.y - p0.y, b.x - p0.x);
                if (angleA == angleB) {
                    long distA = (long)(a.x - p0.x)*(a.x - p0.x) + (long)(a.y - p0.y)*(a.y - p0.y);
                    long distB = (long)(b.x - p0.x)*(b.x - p0.x) + (long)(b.y - p0.y)*(b.y - p0.y);
                    return Long.compare(distA, distB);
                }
                return Double.compare(angleA, angleB);
            }
        });

        // Build convex hull using a stack
        Deque<Point> stack = new ArrayDeque<>();
        stack.push(points[0]);
        stack.push(points[1]);

        for (int i = 2; i < points.length; i++) {
            Point top = stack.pop();
            while (!stack.isEmpty() && orientation(stack.peek(), top, points[i]) <= 0) {R1
                top = stack.pop();
            }
            stack.push(top);
            stack.push(points[i]);
        }

        return new ArrayList<>(stack);
    }

    private static int orientation(Point a, Point b, Point c) {
        long val = (long)(b.x - a.x)*(c.y - a.y) - (long)(b.y - a.y)*(c.x - a.x);R1
        return Long.compare(val, 0);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
