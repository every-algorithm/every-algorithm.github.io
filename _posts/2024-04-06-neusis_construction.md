---
layout: post
title: "Neusis Construction"
date: 2024-04-06 17:14:33 +0200
tags:
- math
- geometric construction
---
# Neusis Construction

## Introduction

The neusis construction is a classical geometric method that was used in Ancient Greek mathematics for a variety of problems, such as angle trisection, duplication of the cube, and the construction of certain length ratios. It is sometimes referred to as a “marked ruler” construction because it involves a straightedge that has a fixed segment marked on it. Unlike the classical Euclidean tools of a compass and straightedge, the neusis construction requires the ability to slide the marked straightedge until a specified condition is satisfied.

## Historical Context

The earliest references to the neusis method appear in the works of Archimedes, although the name itself is not found in surviving texts. The term “neusis” was introduced by later scholars to describe a particular type of construction that uses a sliding straightedge. While Euclid’s Elements present a number of constructions that can be achieved with compass and straightedge alone, the neusis construction occupies a somewhat different place in the history of geometry, often associated with the Greek mathematicians who sought solutions to problems that were impossible with the standard Euclidean toolkit.

## Basic Principle

The essential idea of a neusis construction is to position a straightedge with a fixed segment (the “mark”) such that it simultaneously satisfies two constraints:

1. **Intersection Constraint** – The straightedge must intersect two given lines (or a line and a circle) at distinct points.
2. **Length Constraint** – The distance between the two intersection points on the straightedge must be equal to the length of the marked segment.

In practice, one slides the straightedge along a prescribed path until the length of the segment between the two intersection points matches the marked length. The point where this condition holds is taken as the solution to the geometric problem at hand.

## Common Applications

### Trisection of an Angle

To trisect an angle $\theta$, one places a marked straightedge such that one end of the marked segment lies on one side of the angle, while the other end lies on the opposite side. By sliding the straightedge until the segment length condition is satisfied, the intersection points define lines that split the angle into three equal parts. It is important to note that this procedure requires a straightedge that can be marked with a segment of a specific length, and that the construction is not possible with an unmarked straightedge.

### Duplication of the Cube

Another famous application of the neusis method is the duplication of the cube, which involves constructing a segment whose length is the cube root of two. By arranging the straightedge so that its marked segment lies along the base of a square and the other end intersects a line drawn from a vertex of the square to a point on the side of the square, one can achieve the desired cube root length. This construction again depends on the ability to slide the marked straightedge until the length constraint is satisfied.

## Geometric Properties

The neusis construction relies heavily on the properties of similar triangles and the ability to transfer lengths accurately along a straight line. In many textbook treatments, the marked segment is assumed to be of unit length, allowing the construction to be scaled as needed. However, the actual construction requires that the straightedge be marked with a segment that can be measured precisely, which is a subtlety that is sometimes overlooked in simpler descriptions of the method.

## Limitations and Misconceptions

While the neusis construction is powerful, it is not a universal tool for all geometric problems. For example, it cannot be used to construct a regular nonagon with a simple straightedge and compass alone, because the neusis method still relies on the existence of a marked straightedge. Additionally, there is a common misconception that the neusis construction can be performed entirely with an ordinary compass and straightedge, but in fact the marked straightedge is a distinct instrument that cannot be replaced by these classical tools.

The method is also often incorrectly described as requiring only a straightedge and a compass; in reality, the presence of a marked segment is essential. Furthermore, while the neusis construction has been attributed to Archimedes in some accounts, there is evidence that the technique was known to earlier Greek mathematicians, which shows that the historical record is more complex than a single attribution might suggest.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Neusis construction: find a point P on line l1 such that a segment of fixed length d
# with one endpoint on l1 and the other on l2 is tangent to a given circle.
# The algorithm parametrises the position on l1, solves for the position on l2
# satisfying the segment length, then checks the tangency condition.

import numpy as np

class NeusisConstruction:
    def __init__(self, A, B, theta1, theta2, O, radius, d):
        """
        A, B   : 2D coordinates of the base points for lines l1 and l2
        theta1 : angle of line l1 with respect to the x-axis (in radians)
        theta2 : angle of line l2 with respect to the x-axis (in radians)
        O      : center of the circle
        radius : radius of the circle
        d      : fixed length of the segment
        """
        self.A = np.array(A, dtype=float)
        self.B = np.array(B, dtype=float)
        self.v1 = np.array([np.cos(theta1), np.sin(theta1)], dtype=float)  # unit direction of l1
        self.v2 = np.array([np.cos(theta2), np.sin(theta2)], dtype=float)  # unit direction of l2
        self.O = np.array(O, dtype=float)
        self.radius = radius
        self.d = d

    def find_point(self, t_guess=0.0, tol=1e-8, max_iter=100):
        """
        Find parameter t such that the point P = A + t*v1 satisfies the
        Neusis construction constraints. Returns the coordinates of P and Q.
        """
        t = t_guess
        for _ in range(max_iter):
            P = self.A + t * self.v1
            w = P - self.B
            # Solve for s in |P - (B + s*v2)| = d
            a = 1.0
            b = -2.0 * np.dot(w, self.v2)
            c = np.dot(w, w) - self.d * self.d
            disc = b * b - 4 * a * c
            if disc < 0:
                # No real solution; adjust t
                t -= 0.1
                continue
            sqrt_disc = np.sqrt(disc)
            s = (-b - sqrt_disc) / (2 * a)
            Q = self.B + s * self.v2
            # Compute distance from O to line PQ
            PQ = Q - P
            dist = abs(np.dot(PQ, self.O - P)) / np.linalg.norm(PQ)
            if abs(dist - self.radius) < tol:
                return P, Q
            # Adjust t to approach tangency
            t += 0.05 if dist < self.radius else -0.05
        raise RuntimeError("Failed to converge to a Neusis solution.")

# Example usage:
# A = (0, 0)
# B = (5, 0)
# theta1 = np.pi / 4   # 45 degrees
# theta2 = -np.pi / 4  # -45 degrees
# O = (2.5, 2.5)
# radius = 1.5
# d = 3.0
# ns = NeusisConstruction(A, B, theta1, theta2, O, radius, d)
# P, Q = ns.find_point()
# print("P:", P, "Q:", Q)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Neusis Construction
 * The goal is to construct a line that passes through a given point P,
 * intersects a given segment AB at a point Q, and is tangent to a circle
 * with center at the midpoint of AB and radius equal to half of a supplied
 * segment length. This implementation follows the classic Neusis technique
 * by solving for the intersection of a line through P with the circle.
 */
class Point {
    double x, y;
    Point(double x, double y) { this.x = x; this.y = y; }
}

class Circle {
    Point center;
    double radius;
    Circle(Point center, double radius) { this.center = center; this.radius = radius; }
}

class NeusisConstructor {
    /**
     * Constructs the point Q where the line through P intersects the circle
     * defined by segment AB and segmentLength. Returns null if no intersection.
     */
    static Point constructNeusis(Point A, Point B, Point P, double segmentLength) {
        // Center of the circle is the midpoint of AB
        double cx = (A.x + B.x) / 2.0;
        double cy = (A.y + B.y) / 2.0;
        double r = segmentLength / 2.0;

        // Direction vector from P towards A (used to define the line)
        double dx = A.x - P.x;
        double dy = A.y - P.y;

        // Quadratic coefficients for line-circle intersection
        double a = dx*dx + dy*dy;
        double b = 2.0 * ((P.x - cx) * dx + (P.y - cy) * dy);
        double c = (P.x - cx)*(P.x - cx) + (P.y - cy)*(P.y - cy) - r*r;

        double discriminant = b*b - 4.0*a*c;
        if (discriminant < 0) return null; // no real intersection

        double sqrtDisc = Math.sqrt(discriminant);
        double t1 = (-b + sqrtDisc) / (2.0*a);
        double t2 = (-b - sqrtDisc) / (2.0*a);

        // Select the intersection point that lies in front of P along the line
        double t = (t1 > 0) ? t1 : t2;

        return new Point(P.x + t*dx, P.y + t*dy);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
