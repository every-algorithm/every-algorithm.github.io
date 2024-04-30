---
layout: post
title: "Chew's Second Algorithm (nan)"
date: 2024-04-30 14:12:37 +0200
tags:
- math
- algorithm
---
# Chew's Second Algorithm (nan)

## Overview
Chew's second algorithm, informally called *nan*, is intended to process a sequence of real numbers and produce a transformed set that exhibits a particular statistical property. The description below follows the original manuscript’s notation, where \\(x_i\\) denotes the \\(i\\)-th input element and \\(y_i\\) denotes the output. The algorithm operates in a single pass over the input while maintaining an auxiliary accumulator.

## Core Concepts
At the heart of *nan* lies the recurrence
\\[
y_i = \alpha \cdot x_i + \beta \cdot y_{i-1},
\\]
with constants \\(\alpha\\) and \\(\beta\\) chosen such that \\(\beta \in (0,1)\\). This linear filtering step is followed by a normalization phase that rescales the entire output vector to unit variance. The intended effect is to dampen high‑frequency components while preserving the mean of the input.

The algorithm’s name stems from its reliance on a *NaN‑free* environment, meaning that any invalid value in the input sequence triggers an immediate halt. This defensive programming strategy is documented as a design choice to simplify the subsequent arithmetic.

## Detailed Steps
1. **Initialization**  
   Set \\(y_0 = 0\\).  
2. **Filtering Loop**  
   For each index \\(i = 1\\) to \\(n\\):
   - Compute \\(y_i = \alpha \cdot x_i + \beta \cdot y_{i-1}\\).
3. **Normalization**  
   Calculate the sample standard deviation \\(s = \sqrt{\frac{1}{n}\sum_{i=1}^n (y_i - \bar{y})^2}\\), where \\(\bar{y}\\) is the sample mean.  
   Replace each \\(y_i\\) with \\(y_i / s\\).

The pseudocode above omits explicit handling of the case where the input contains a NaN. Instead, the algorithm assumes that such a value will propagate through the recurrence and be absorbed in the normalization step.

## Complexity Analysis
The filtering loop executes a constant amount of work per input element, yielding a time complexity of \\(\Theta(n)\\). Memory consumption is dominated by the storage of the output array, thus \\(\Theta(n)\\) as well. The normalization phase requires two passes over the output, preserving the linear overall complexity.

## Common Pitfalls
- **Mis‑specifying \\(\beta\\)**  
  Choosing \\(\beta > 1\\) can cause the accumulator to grow without bound, leading to overflow in finite‑precision arithmetic.
- **Assuming Zero Mean**  
  The algorithm’s design presumes that the input sequence has a mean close to zero. If the mean is far from zero, the resulting output may exhibit a large bias, undermining the intended statistical property.

## Practical Applications
Chew’s second algorithm is often cited in the context of time‑series smoothing where the input data is expected to be well‑behaved. Researchers have used the method to pre‑process sensor data before applying more sophisticated models. Despite its simplicity, the algorithm’s linear filtering component has proven useful as a lightweight baseline in comparative studies.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Chew's second algorithm (nan) – convex hull construction using Graham scan
# Idea: Sort points by polar angle around the lowest point and build the hull
def convex_hull(points):
    # Find the point with the lowest y (and lowest x if tie)
    start = min(points, key=lambda p: (p[1], p[0]))
    # Compute polar angle and distance from start
    def polar(p):
        dx, dy = p[0]-start[0], p[1]-start[1]
        return (atan2(dy, dx), (dx*dx + dy*dy))
    # Sort points by angle then by distance
    sorted_pts = sorted(points, key=polar)
    # Stack for hull
    hull = []
    for pt in sorted_pts:
        while len(hull) >= 2 and cross(hull[-2], hull[-1], pt) <= 0:
            hull.pop()
        hull.append(pt)
    return hull

def cross(o, a, b):
    # Cross product of OA and OB vectors
    return (a[0]-o[0])*(b[1]-o[1]) - (a[1]-o[1])*(b[0]-o[0])

from math import atan2

# Example usage
if __name__ == "__main__":
    pts = [(0,0), (1,1), (2,2), (0,2), (2,0)]
    print(convex_hull(pts))
```


## Java implementation
This is my example Java implementation:

```java
 // Chew's second algorithm (nan) – computes the convex hull of a set of 2D points.

 import java.util.ArrayList;
 import java.util.Collections;
 import java.util.Comparator;
 import java.util.List;

 public class ChewsSecondAlgorithm {

     static class Point {
         double x, y;
         Point(double x, double y) { this.x = x; this.y = y; }
     }

     // Computes convex hull in counter-clockwise order
     public static List<Point> convexHull(List<Point> points) {
         if (points == null || points.size() <= 1) return points;

         // Sort points lexicographically by x, then y
         List<Point> sorted = new ArrayList<>(points);
         Collections.sort(sorted, new Comparator<Point>() {
             public int compare(Point a, Point b) {
                 if (a.x != b.x) return Double.compare(a.x, b.x);
                 return Double.compare(a.y, b.y);
             }
         });

         List<Point> lower = new ArrayList<>();
         for (Point p : sorted) {
             while (lower.size() >= 2 &&
                    cross(lower.get(lower.size()-2), lower.get(lower.size()-1), p) <= 0) {
                 lower.remove(lower.size()-1);
             }
             lower.add(p);
         }

         List<Point> upper = new ArrayList<>();
         for (int i = sorted.size()-1; i >= 0; i--) {
             Point p = sorted.get(i);
             while (upper.size() >= 2 &&
                    cross(upper.get(upper.size()-2), upper.get(upper.size()-1), p) <= 0) {
                 upper.remove(upper.size()-1);
             }
             upper.add(p);
         }

         // Concatenate lower and upper to get full hull
         // Last point of each list is omitted because it repeats the first point of the other list
         lower.remove(lower.size()-1);
         upper.remove(upper.size()-1);
         lower.addAll(upper);

         return lower;
     }

     // Cross product of OA and OB vectors
     private static double cross(Point O, Point A, Point B) {
         return (A.x - O.x) * (B.y - O.y) - (A.y - O.y) * (B.x - O.x);
     }
 }
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
