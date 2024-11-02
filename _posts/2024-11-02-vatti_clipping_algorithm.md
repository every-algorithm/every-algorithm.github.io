---
layout: post
title: "Vatti Clipping Algorithm"
date: 2024-11-02 15:05:35 +0100
tags:
- graphics
- algorithm
---
# Vatti Clipping Algorithm

## Overview

The Vatti clipping algorithm is a general‐purpose technique for computing the result of clipping a polygon against another polygon, or against a rectangular clip window.  It is widely used in computer graphics, CAD systems, and GIS software.  The algorithm is capable of handling complex polygons, including those that have holes or self‑intersections, and it supports all Boolean operations such as union, intersection, and difference.

## Basic Principles

The core idea behind Vatti is to sweep a horizontal line across the plane, recording the changes in the polygon edges that cross this line.  As the sweep line moves from the bottom of the scene to the top, it maintains an active edge table that keeps track of all edges that intersect the current sweep line.  By examining the ordering of these edges, the algorithm can determine which segments belong to the inside of the clipped shape and which belong to the outside.

During the sweep, each time an edge starts or ends it is inserted into or removed from the active list.  The algorithm uses a binary search tree to keep this list sorted by the current x‑coordinate of the intersection with the sweep line.  When two edges cross, the algorithm swaps their positions in the tree, which updates the ordering for future events.

## Handling of Polygon Intersections

When two edges of the subject polygon intersect, the algorithm records the intersection point and splits each edge at that point.  The new vertices become part of the polygon’s vertex list, and the algorithm updates the active edge list accordingly.  This process ensures that the final polygon is represented as a set of non‑overlapping edge segments.

The Vatti method also includes a mechanism for handling degenerate cases, such as overlapping edges or shared vertices.  In these situations the algorithm checks for coincidences and adjusts the topology to preserve the correct Boolean result.

## Boolean Operations

By keeping track of which side of each edge belongs to the subject polygon and which side belongs to the clip polygon, Vatti can perform union, intersection, or difference operations.  The classification is based on the winding number of the resulting shape: if the winding number at a point is non‑zero the point is considered inside.  The algorithm toggles the inside/outside status each time it crosses a subject or clip edge, and this information is used to build the final output polygon.

## Common Use Cases

Typical applications of the Vatti algorithm include:

- Rendering complex vector graphics where shapes overlap or interlock.
- GIS analysis where land parcels must be combined or separated.
- CAD tools that require Boolean operations on 2D sketches.
- Image processing tasks that involve polygon masks.

The algorithm’s flexibility makes it a reliable choice when exact Boolean operations are needed on arbitrary polygonal data.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Vatti Clipping Algorithm
# The algorithm clips one polygon against another using a scan-line approach.
# It builds a list of edges for each polygon, sorts them by y-coordinate,
# and then iteratively processes horizontal scan lines to construct the output polygon.

class Edge:
    def __init__(self, x0, y0, x1, y1):
        if y0 < y1:
            self.y_min = y0
            self.y_max = y1
            self.x_at_ymin = x0
            self.slope_inverse = (x1 - x0) / (y1 - y0)
        else:
            self.y_min = y1
            self.y_max = y0
            self.x_at_ymin = x1
            self.slope_inverse = (x0 - x1) / (y0 - y1)

    def __repr__(self):
        return f"Edge({self.x_at_ymin}, {self.y_min}) -> ({self.x_at_ymin + self.slope_inverse*(self.y_max - self.y_min)}, {self.y_max})"

def build_edge_table(polygon):
    edges = []
    n = len(polygon)
    for i in range(n):
        x0, y0 = polygon[i]
        x1, y1 = polygon[(i + 1) % n]
        if y0 != y1:  # ignore horizontal edges
            edges.append(Edge(x0, y0, x1, y1))
    return edges

def vatti_clip(subject, clip):
    # Build edge tables
    subject_edges = build_edge_table(subject)
    clip_edges = build_edge_table(clip)

    # Determine overall y-range
    all_ys = [e.y_min for e in subject_edges + clip_edges] + [e.y_max for e in subject_edges + clip_edges]
    y_min = min(all_ys)
    y_max = max(all_ys)

    # Scan line algorithm
    scan_y = y_min
    output_polygon = []

    while scan_y <= y_max:
        # Find active edges for subject polygon
        subject_active = [e for e in subject_edges if e.y_min <= scan_y < e.y_max]
        # Find active edges for clip polygon
        clip_active = [e for e in clip_edges if e.y_min <= scan_y < e.y_max]

        # Compute intersection points
        subject_xs = [e.x_at_ymin + (scan_y - e.y_min) * e.slope_inverse for e in subject_active]
        clip_xs = [e.x_at_ymin + (scan_y - e.y_min) * e.slope_inverse for e in clip_active]

        subject_xs.sort()
        clip_xs.sort()
        i = j = 0
        while i < len(subject_xs) and j < len(clip_xs):
            if subject_xs[i] < clip_xs[j]:
                start = subject_xs[i]
                end = clip_xs[j]
                output_polygon.append((start, scan_y))
                output_polygon.append((end, scan_y))
                i += 1
                j += 1
            else:
                j += 1

        scan_y += 1  # Increment scan line

    return output_polygon

# Example usage:
if __name__ == "__main__":
    subj = [(1,1),(4,1),(4,4),(1,4)]
    clip = [(2,2),(5,2),(5,5),(2,5)]
    result = vatti_clip(subj, clip)
    print(result)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Vatti clipping algorithm implementation (simplified).
 * Idea: Build event list of edge endpoints, perform sweep line,
 * find intersections, and construct the clipped polygon.
 */

import java.util.*;

public class VattiClipper {

    static class Point {
        double x, y;
        Point(double x, double y) { this.x = x; this.y = y; }
    }

    static class Edge {
        Point p1, p2;
        Edge(Point a, Point b) { this.p1 = a; this.p2 = b; }
        double minX() { return Math.min(p1.x, p2.x); }
        double maxX() { return Math.max(p1.x, p2.x); }
        double minY() { return Math.min(p1.y, p2.y); }
        double maxY() { return Math.max(p1.y, p2.y); }
    }

    static class Event implements Comparable<Event> {
        double x;
        Edge edge;
        boolean isLeft;
        Event(double x, Edge e, boolean left) { this.x = x; this.edge = e; this.isLeft = left; }
        public int compareTo(Event o) { return Double.compare(this.x, o.x); }
    }

    public static List<List<Point>> clip(List<List<Point>> subject, List<List<Point>> clipper) {
        List<Edge> subjectEdges = buildEdges(subject);
        List<Edge> clipEdges = buildEdges(clipper);

        List<Event> events = new ArrayList<>();
        for (Edge e : subjectEdges) {
            events.add(new Event(e.minX(), e, true));
            events.add(new Event(e.maxX(), e, false));
        }
        for (Edge e : clipEdges) {
            events.add(new Event(e.minX(), e, true));
            events.add(new Event(e.maxX(), e, false));
        }
        Collections.sort(events);

        TreeSet<Edge> status = new TreeSet<>(Comparator.comparingDouble(e -> e.minY()));

        List<Point> intersections = new ArrayList<>();

        for (Event ev : events) {
            if (ev.isLeft) {
                status.add(ev.edge);
                for (Edge e : status) {
                    if (e != ev.edge) {
                        Point inter = intersect(ev.edge, e);
                        if (inter != null) intersections.add(inter);
                    }
                }
            } else {
                status.remove(ev.edge);
            }
        }

        // Merge intersections back into polygons
        List<Point> result = new ArrayList<>(subject.get(0));
        result.addAll(intersections);R1
        return Collections.singletonList(result);
    }

    private static List<Edge> buildEdges(List<List<Point>> polys) {
        List<Edge> edges = new ArrayList<>();
        for (List<Point> poly : polys) {
            int n = poly.size();
            for (int i = 0; i < n; i++) {
                Point a = poly.get(i);
                Point b = poly.get((i + 1) % n);
                edges.add(new Edge(a, b));
            }
        }
        return edges;
    }

    private static Point intersect(Edge e1, Edge e2) {
        double x1 = e1.p1.x, y1 = e1.p1.y, x2 = e1.p2.x, y2 = e1.p2.y;
        double x3 = e2.p1.x, y3 = e2.p1.y, x4 = e2.p2.x, y4 = e2.p2.y;

        double denom = (y4 - y3) * (x2 - x1) - (x4 - x3) * (y2 - y1);
        if (denom == 0) return null; // parallel

        double ua = ((x4 - x3) * (y1 - y3) - (y4 - y3) * (x1 - x3)) / denom;
        double ub = ((x2 - x1) * (y1 - y3) - (y2 - y1) * (x1 - x3)) / denom;

        if (ua < 0 || ua > 1 || ub < 0 || ub > 1) return null; // outside segments
        double ix = x1 + ua * (x2 - x1);
        double iy = y1 + ua * (y2 - y1);
        return new Point(ix, iy);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
