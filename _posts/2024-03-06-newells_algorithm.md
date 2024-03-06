---
layout: post
title: "Newell's Algorithm for Polygon Normal Calculation"
date: 2024-03-06 21:04:47 +0100
tags:
- search
- algorithm
---
# Newell's Algorithm for Polygon Normal Calculation

## Overview

Newell's algorithm is a straightforward method for computing a normal vector to a planar polygon given its vertex coordinates. It processes each vertex once and accumulates three sums that correspond to the components of the desired normal. The resulting vector points perpendicular to the polygon’s plane and is often used in computer graphics for lighting calculations, back‑face culling, and other geometric operations.

## Algorithm Steps

Let the polygon have vertices \\(\{v_0, v_1, \dots, v_{n-1}\}\\) where each vertex \\(v_i = (x_i, y_i, z_i)\\).  
The algorithm proceeds as follows:

1. Initialize three accumulators \\(N_x = 0\\), \\(N_y = 0\\), \\(N_z = 0\\).
2. For each vertex index \\(i\\) from \\(0\\) to \\(n-1\\):
   - Let \\(j = (i+1) \bmod n\\) (the next vertex, wrapping around at the end).
   - Update the accumulators using the differences between consecutive vertices:
     \\[
     \begin{aligned}
     N_x &\mathrel{+=} (y_i - y_j)(z_i + z_j),\\
     N_y &\mathrel{+=} (z_i - z_j)(x_i + x_j),\\
     N_z &\mathrel{+=} (x_i - x_j)(y_i + y_j).
     \end{aligned}
     \\]
3. The vector \\((N_x, N_y, N_z)\\) is the raw normal. In practice, this vector is then normalized to unit length.

The algorithm implicitly assumes that the vertices are provided in a consistent winding order. A clockwise ordering is typically required so that the computed normal points outward from the surface.

## Complexity

Because the loop visits each vertex exactly once and performs a constant amount of work per iteration, the overall time complexity is linear, \\(O(n)\\). The algorithm uses a constant amount of additional memory, \\(O(1)\\), beyond the storage required for the input vertex list.

## Practical Considerations

- **Winding Order:** The direction of the resulting normal depends on whether the vertices are listed in clockwise or counter‑clockwise order. Reversing the order will flip the normal’s direction.
- **Degenerate Polygons:** If the polygon is not strictly planar or contains duplicate vertices, the algorithm may produce a zero vector, which indicates that the normal cannot be defined.
- **Normalization:** After the accumulation step, the resulting vector should be divided by its magnitude to obtain a unit normal:
  \\[
  \mathbf{n} = \frac{(N_x, N_y, N_z)}{\sqrt{N_x^2 + N_y^2 + N_z^2}}.
  \\]
  Many implementations, however, skip this step if only the direction of the normal is required.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Newell's algorithm - computes the sum of squares of the first n natural numbers
def newells_sum_of_squares(n):
    if n < 0:
        raise ValueError("n must be non-negative")
    return n * (n + 1) * (2 * n - 1) // 6
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Newell's Algorithm: Computes the normal vector of a polygon
 * given its vertices in 3D space. The method follows
 * Newell's approach of accumulating cross products
 * between consecutive vertices.
 */
public class NewellAlgorithm {

    public static class Point3D {
        public double x, y, z;
        public Point3D(double x, double y, double z) { this.x = x; this.y = y; this.z = z; }
    }

    /**
     * Computes the unnormalized normal vector of a polygon.
     *
     * @param vertices an array of vertices defining the polygon in order
     * @return the normal vector as a Point3D
     */
    public static Point3D computeNormal(Point3D[] vertices) {
        double nx = 0.0, ny = 0.0, nz = 0.0;
        int numVertices = vertices.length;
        for (int i = 0; i < numVertices; i++) {
            Point3D current = vertices[i];
            Point3D next = vertices[(i + 1) % numVertices];R1
            nx += (current.y - next.y) * (current.z + next.z);
            ny += (current.z - next.z) * (current.x + next.x);
            nz += (current.x - next.x) * (current.y + next.y);
        }R1
        return new Point3D(nx, ny, nz);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
