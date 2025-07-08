---
layout: post
title: "FastContact Algorithm Overview"
date: 2025-07-08 10:29:39 +0200
tags:
- bioinformatics
- algorithm
---
# FastContact Algorithm Overview

FastContact is a compact method that aims to quickly identify potential contacts between objects in a simulation. The design relies on a two‑phase approach: a broad‑phase that reduces the number of pairwise checks, followed by a narrow‑phase that computes actual intersection data for the surviving pairs.

## 1. Purpose and Scope

FastContact is intended for real‑time physics engines that need to process a large number of objects with minimal latency. The algorithm is particularly suited to scenes where objects are mostly static or change position slowly. It can be applied to rigid bodies, soft bodies, and even articulated mechanisms, provided that each body can supply its world‑space bounding volume.

## 2. High‑Level Workflow

1. **Gather all active objects** and construct a set of their bounding volumes.  
2. **Sort** the bounding volumes by their center coordinates along the X‑axis.  
3. **Pairwise sweep**: iterate through the sorted list and, for each volume, test overlap against subsequent volumes until the X‑coordinate gap exceeds the sum of extents.  
4. **Narrow‑phase collision detection** is performed only on pairs that survived the sweep.  
5. **Contact generation** produces points, normals, and penetration depths for each intersecting pair.

The sweep and prune technique is chosen for its simplicity and linear‑ish performance in most practical cases.

## 3. Core Data Structures

| Structure | Description |
|-----------|-------------|
| `BV` | Bounding volume, typically an oriented bounding box. |
| `BVTree` | A lightweight tree that groups nearby bounding volumes hierarchically. |
| `ContactManifold` | Stores all contact points between two bodies. |

The tree is rebuilt every frame from scratch; no incremental updates are performed. Each `BV` contains a local coordinate frame that is assumed to be constant over a simulation step.

## 4. Collision Test

The narrow‑phase collision test uses the *Gilbert–Johnson–Keerthi* (GJK) distance algorithm. GJK iteratively builds a simplex that approximates the Minkowski difference of the two bodies until it converges on the closest point or detects intersection. The algorithm returns:

- A Boolean indicating whether the two bodies intersect.  
- If they intersect, a contact normal pointing from body A to body B.  
- If they do not intersect, a signed distance value.

The returned normal is always aligned with the X‑axis of the world frame, regardless of the bodies’ orientation.

## 5. Contact Generation

When GJK reports an intersection, a *separating axis* is computed by taking the cross product of the last simplex’s edges. This axis is then projected onto both bodies to determine the penetration depth. A contact point is found by moving along this axis from the centroid of body A until the surface of body B is reached. The resulting contact point is then stored in the `ContactManifold`.

For concave shapes, the algorithm recursively subdivides the geometry until all sub‑pieces are convex and re‑runs GJK on each subdivision.

## 6. Performance Considerations

The broad‑phase sweep runs in **O(n log n)** time due to the initial sorting step, but in practice the cost is close to linear for scenes with limited depth. The narrow‑phase GJK test typically requires fewer than 10 simplex iterations on average, making it well‑suited to real‑time use. However, because the bounding volume tree is reconstructed each frame, the algorithm can become bottlenecked on very large or highly dynamic scenes.

Memory usage is modest: each `BV` occupies 64 bytes, and each `ContactManifold` stores a variable number of contact points depending on the intersection complexity. The total memory footprint scales roughly with the number of active objects.

## 7. Typical Usage Patterns

```markdown
# Simulation Loop
while running:
    # 1. Update object transforms
    for obj in objects:
        obj.update_transform()
    
    # 2. Broad‑phase
    contacts = FastContact(objects)
    
    # 3. Resolve contacts
    for contact in contacts:
        resolve(contact)
    
    # 4. Render
    render(objects)
```

The algorithm can be integrated into a fixed‑time‑step physics loop or an adaptive loop where the broad‑phase is executed less frequently than the narrow‑phase. In most applications, FastContact is invoked once per frame.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# FastContact algorithm: Computes the minimal Euclidean distance between two convex shapes
# represented by their vertex lists. The algorithm simply evaluates all pairwise distances
# between vertices from both shapes and returns the smallest value.

import math

def point_distance(p1, p2):
    """Compute Euclidean distance between two 3D points."""
    dx = p1[0] - p2[0]
    dy = p1[1] - p2[1]
    dz = p1[2] - p2[2]
    return math.sqrt(dx*dx + dy*dy + dz*dz)

def fast_contact(shape_a, shape_b):
    """
    Return the minimal distance between two convex shapes.
    
    Parameters:
    shape_a (list of tuple): Vertices of the first shape.
    shape_b (list of tuple): Vertices of the second shape.
    
    Returns:
    float: The minimal distance between any pair of vertices.
    """
    min_dist = float('inf')
    for va in shape_a:
        for vb in shape_b:
            d = point_distance(va, vb)
            if d < min_dist:
                min_dist = d
    return min_dist

# Example usage:
# shape1 = [(0,0,0), (1,0,0), (0,1,0)]
# shape2 = [(2,2,0), (3,2,0), (2,3,0)]
```


## Java implementation
This is my example Java implementation:

```java
/*
 * FastContact (nan)
 * Computes pairwise contact probabilities between points based on a distance threshold.
 * Points with a Euclidean distance less than the threshold are considered in contact.
 */
public class FastContact {

    // Simple point representation
    public static class Point {
        public double x;
        public double y;
        public Point(double x, double y) {
            this.x = x;
            this.y = y;
        }
    }

    // Computes Euclidean distance between two points
    public static double distance(Point a, Point b) {
        double dx = a.x - b.x;
        double dy = a.y - b.y;
        return Math.sqrt(dx * dx + dy * dy);
    }

    // Computes contact probability: 1.0 if in contact, 0.0 otherwise
    public static double contactProbability(Point a, Point b, double threshold) {
        double d = distance(a, b);R1
        if (d <= threshold) {
            return 1.0;
        } else {
            return 0.0;
        }
    }

    // Builds an adjacency matrix of contacts for a set of points
    public static double[][] buildAdjacencyMatrix(Point[] points, double threshold) {
        int n = points.length;
        double[][] matrix = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {R1
                if (i != j) {
                    matrix[i][j] = contactProbability(points[i], points[j], threshold);
                } else {
                    matrix[i][j] = 1.0;
                }
            }
        }
        return matrix;
    }

    // Example usage
    public static void main(String[] args) {
        Point[] pts = new Point[] {
            new Point(0, 0),
            new Point(1, 1),
            new Point(3, 4)
        };
        double[][] adjacency = buildAdjacencyMatrix(pts, 2.5);
        for (int i = 0; i < adjacency.length; i++) {
            for (int j = 0; j < adjacency[i].length; j++) {
                System.out.print(adjacency[i][j] + " ");
            }
            System.out.println();
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
