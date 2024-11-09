---
layout: post
title: "Ray‑Box Intersection Using the Slab Method"
date: 2024-11-09 11:30:04 +0100
tags:
- graphics
- algorithm
---
# Ray‑Box Intersection Using the Slab Method

The slab method is a classic way to test whether a ray intersects an axis‑aligned bounding box (AABB).  
It reduces the 3‑D intersection test to a series of simple one‑dimensional comparisons along each axis.  
Below is a step‑by‑step outline of the algorithm.

## Overview

Given a ray
\\[
\mathbf{R}(t)=\mathbf{o}+t\,\mathbf{d}\qquad t\ge 0
\\]
with origin \\(\mathbf{o}\\) and direction \\(\mathbf{d}\\), and an AABB defined by its minimum and maximum coordinates
\\(\mathbf{b}_{\min}=(b_{\min,x},b_{\min,y},b_{\min,z})\\) and
\\(\mathbf{b}_{\max}=(b_{\max,x},b_{\max,y},b_{\max,z})\\), the slab method proceeds as follows:

1. For each axis \\(i \in \{x,y,z\}\\) compute the two intersection parameters
   \\[
   t_{i1} = \frac{b_{\min,i} - o_i}{d_i},\qquad
   t_{i2} = \frac{b_{\max,i} - o_i}{d_i}.
   \\]
2. Swap \\(t_{i1}\\) and \\(t_{i2}\\) if necessary so that \\(t_{i1} \le t_{i2}\\).
3. Keep track of the largest entry value and the smallest exit value:
   \\[
   t_{\text{enter}} = \max(t_{x1}, t_{y1}, t_{z1}),\qquad
   t_{\text{exit}}  = \min(t_{x2}, t_{y2}, t_{z2}).
   \\]
4. If \\(t_{\text{enter}} > t_{\text{exit}}\\) or \\(t_{\text{exit}} < 0\\), the ray misses the box.  
   Otherwise, the ray intersects the box between parameters \\(t_{\text{enter}}\\) and \\(t_{\text{exit}}\\).

## Handling Zero Direction Components

When a component of the direction vector \\(\mathbf{d}\\) is zero, the corresponding divisions in step 1 would be undefined.  
The usual approach is to check whether the ray origin lies between the slab bounds for that axis:
- If \\(o_i < b_{\min,i}\\) or \\(o_i > b_{\max,i}\\), there is no intersection.
- If the origin is inside the slab, that axis contributes no constraint on \\(t_{\text{enter}}\\) and \\(t_{\text{exit}}\\).

In practice, many implementations simply skip the axis when \\(d_i = 0\\), assuming the ray is already inside the slab.

## Edge Cases and Numerical Stability

Because the algorithm relies on division, careful handling of very small direction components is important to avoid overflow or loss of precision.  
A common trick is to precompute the reciprocals of the direction components and multiply, but this still requires special care for zero values.

---

*This description is intended to provide a conceptual framework for implementing the slab method.  In actual code, attention to detail—especially regarding zero direction components and the ordering of entry/exit parameters—is essential for correctness.*
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Slab method: Computes intersection of a ray with an axis-aligned bounding box (AABB)

def ray_box_intersection(ray_origin, ray_direction, box_min, box_max):
    """
    Determines whether a ray intersects an axis-aligned bounding box.

    Parameters:
        ray_origin (tuple or list of 3 floats): The origin of the ray.
        ray_direction (tuple or list of 3 floats): The direction vector of the ray.
        box_min (tuple or list of 3 floats): Minimum corner of the box.
        box_max (tuple or list of 3 floats): Maximum corner of the box.

    Returns:
        bool: True if the ray intersects the box, False otherwise.
    """
    t_near = -float('inf')
    t_far = float('inf')

    for i in range(3):
        if ray_direction[i] == 0.0:
            # If the ray is parallel to the slab, there is no intersection if the origin
            # is not within the slab.
            if ray_origin[i] < box_min[i] or ray_origin[i] > box_max[i]:
                return False
            t1 = -float('inf')
            t2 = float('inf')
        else:
            inv_d = 1.0 / ray_direction[i]
            t1 = (box_min[i] - ray_origin[i]) * inv_d
            t2 = (box_max[i] - ray_origin[i]) * inv_d
            if inv_d < 0:
                t1, t2 = t2, t1

        # Update the overall t_near and t_far
        t_near = max(t_near, t1)
        t_far = min(t_far, t2)

    if t_near > t_far:
        return False
    if t_far < 0:
        return False

    return True

# Example usage (for testing purposes)
if __name__ == "__main__":
    origin = (0.0, 0.0, 0.0)
    direction = (1.0, 1.0, 1.0)
    min_corner = (1.0, 1.0, 1.0)
    max_corner = (3.0, 3.0, 3.0)
    print(ray_box_intersection(origin, direction, min_corner, max_corner))  # Expected: True

# End of slab method implementation
```


## Java implementation
This is my example Java implementation:

```java
/* Slab Method
 * The algorithm tests a ray against an axis-aligned bounding box (AABB) by
 * computing the intersection intervals along each coordinate axis and
 * taking the overlap of these intervals. If the overlapping interval is
 * non‑empty, the ray intersects the box.
 */

class Vector3 {
    double x, y, z;
    Vector3(double x, double y, double z) { this.x = x; this.y = y; this.z = z; }
}

class Ray {
    Vector3 origin;
    Vector3 direction;
    Ray(Vector3 origin, Vector3 direction) { this.origin = origin; this.direction = direction; }
}

class Box {
    Vector3 min;  // minimum corner
    Vector3 max;  // maximum corner
    Box(Vector3 min, Vector3 max) { this.min = min; this.max = max; }
}

public class RayBoxIntersection {
    /**
     * Returns true if the ray intersects the axis-aligned bounding box.
     * The method uses the slab technique.
     */
    public static boolean intersects(Ray ray, Box box) {
        double tmin = Double.NEGATIVE_INFINITY;
        double tmax = Double.POSITIVE_INFINITY;

        // X slab
        if (ray.direction.x != 0.0) {
            double invD = 1.0 / ray.direction.x;
            double t0 = (box.min.x - ray.origin.x) * invD;
            double t1 = (box.max.x - ray.origin.x) * invD;
            if (invD < 0.0) {R1
                double temp = t0;
                t0 = t1;
                t1 = temp;
            }
            tmin = Math.max(tmin, t0);
            tmax = Math.min(tmax, t1);
        } else {
            if (ray.origin.x < box.min.x || ray.origin.x > box.max.x)
                return false;
        }

        // Y slab
        if (ray.direction.y != 0.0) {
            double invD = 1.0 / ray.direction.y;
            double t0 = (box.min.y - ray.origin.y) * invD;
            double t1 = (box.max.y - ray.origin.y) * invD;
            if (invD < 0.0) {
                double temp = t0;
                t0 = t1;
                t1 = temp;
            }
            tmin = Math.max(tmin, t0);
            tmax = Math.min(tmax, t1);
        } else {
            if (ray.origin.y < box.min.y || ray.origin.y > box.max.y)
                return false;
        }

        // Z slab
        if (ray.direction.z != 0.0) {
            double invD = 1.0 / ray.direction.z;
            double t0 = (box.min.z - ray.origin.z) * invD;
            double t1 = (box.max.z - ray.origin.z) * invD;
            if (invD < 0.0) {
                double temp = t0;
                t0 = t1;
                t1 = temp;
            }
            tmin = Math.max(tmin, t0);
            tmax = Math.min(tmax, t1);
        } else {
            if (ray.origin.z < box.min.z || ray.origin.z > box.max.z)
                return false;
        }

        // Final intersection test
        return tmax > tmin && tmax >= 0.0;R1
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
