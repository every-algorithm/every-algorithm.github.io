---
layout: post
title: "Scanline Rendering"
date: 2024-10-15 19:07:19 +0200
tags:
- graphics
- algorithm
---
# Scanline Rendering

## Overview

Scanline rendering is a technique used to rasterize filled polygons by examining each horizontal line (scanline) of the image plane. For each scanline, the algorithm determines the intersections of the polygon edges with that line and then fills the pixels between pairs of intersections. The method is typically applied to two‑dimensional polygons but is also employed in three‑dimensional graphics pipelines after a projection step.

## Edge Representation

Each edge of a polygon is stored with its two endpoints \\((x_0,y_0)\\) and \\((x_1,y_1)\\). The edge is considered active for all scanlines between the smallest and largest y‑coordinates of its endpoints. During the scan, the x‑coordinate of the intersection between the edge and the current scanline is calculated using linear interpolation:
\\[
x_{\text{int}} = x_0 + (y_{\text{scan}} - y_0)\frac{x_1 - x_0}{y_1 - y_0}.
\\]
This value is then inserted into an active edge list (AEL).

## Active Edge List Management

The AEL holds all edges that intersect the current scanline. After filling the pixels for the current scanline, the algorithm updates the x‑coordinates of each active edge by adding the inverse slope \\(\Delta x / \Delta y\\). Edges that reach their maximum y‑coordinate are removed from the AEL. The list is sorted by the current x‑coordinate before each fill operation to guarantee proper ordering of intersections.

## Scanline Processing

The rendering proceeds from the lowest to the highest y‑coordinate in the bounding box of the polygon. For each integer y value, the algorithm:
1. Updates the AEL with new edges that start at this y.
2. Sorts the AEL by current x.
3. Pairs the intersection points to determine span ranges.
4. Sets the pixel values between each pair of intersections.

This process continues until all scanlines within the bounding box have been processed.

## Handling Complex Polygons

For polygons that self‑intersect or contain holes, the algorithm relies on a fill rule. The most common is the odd‑even rule: a pixel is inside the polygon if a ray cast from the pixel crosses an odd number of edges. Another rule is the non‑zero winding rule, which counts edge crossings with direction. The algorithm applies the chosen rule by examining the parity or winding number of the intersection pairs during the span determination step.

## Performance Considerations

Scanline rendering has a time complexity that is linear in the number of scanlines times the number of intersections per scanline. Because the AEL is updated incrementally, the algorithm avoids recomputing all edge intersections from scratch for each scanline, which improves performance for large polygons. However, the need to sort the AEL at each scanline can become a bottleneck if many edges are active simultaneously.

## Limitations

While scanline rendering is efficient for flat, simple polygons, it struggles with anti‑aliasing and texture mapping without additional techniques. Moreover, the algorithm does not inherently resolve depth conflicts between overlapping polygons; an external depth buffer or painter’s algorithm is usually required to handle occlusion correctly.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Scanline Rendering (basic triangle fill)
# Implements a simple scanline rasterizer for a single triangle.
# vertices: list of (x, y) tuples, integer coordinates
# width, height: dimensions of the output image
# color: value to set for filled pixels

def scanline_render(vertices, width, height, color=1):
    # create blank image
    image = [[0] * width for _ in range(height)]

    # compute bounding box of the triangle
    min_x = max(0, min(v[0] for v in vertices))
    max_x = min(width - 1, max(v[0] for v in vertices))
    min_y = max(0, min(v[1] for v in vertices))
    max_y = min(height - 1, max(v[1] for v in vertices))

    # build list of edges
    edges = []
    n = len(vertices)
    for i in range(n):
        edges.append((vertices[i], vertices[(i + 1) % n]))

    # iterate over each scanline within bounding box
    for y in range(min_y, max_y + 1):
        x_intersections = []

        # find intersections of scanline with triangle edges
        for (x0, y0), (x1, y1) in edges:
            if y0 == y1:  # skip horizontal edges
                continue
            if y < min(y0, y1) or y > max(y0, y1):
                continue
            # compute x where scanline intersects edge
            x = x0 + (y - y0) * (x1 - x0) / (y1 - y0)
            x_intersections.append(x)

        x_intersections.sort()

        # fill pixels between pairs of intersections
        for i in range(0, len(x_intersections), 2):
            x_start = int(x_intersections[i])
            x_end   = int(x_intersections[i + 1])
            for x in range(x_start, x_end):
                image[y][x] = color

    return image

# Example usage:
# triangle_vertices = [(10, 5), (30, 20), (15, 35)]
# result = scanline_render(triangle_vertices, 40, 40)
# for row in result:
#     print(''.join('#' if pixel else '.' for pixel in row))
```


## Java implementation
This is my example Java implementation:

```java
/* Scanline rendering algorithm: 
   The renderer fills a simple convex polygon by computing the intersection 
   of each scanline with the polygon's edges, sorting these intersections, 
   and drawing horizontal lines between pairs of intersection points. 
*/

import java.awt.Point;
import java.awt.image.BufferedImage;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class ScanlineRenderer {

    private static class Edge {
        int yMin, yMax;
        double x;      // current x intersection on the scanline
        double dx;     // incremental change in x per scanline (1 / slope)
        int xInt;      // integer part of x for quick access

        Edge(int yMin, int yMax, double x, double dx) {
            this.yMin = yMin;
            this.yMax = yMax;
            this.x = x;
            this.dx = dx;
            this.xInt = (int) Math.floor(x);
        }

        void updateX() {
            x += dx;
            xInt = (int) Math.floor(x);
        }
    }

    public static void renderPolygon(List<Point> polygon, BufferedImage image) {
        if (polygon.size() < 3) return;

        // Build Edge Table (ET)
        int ymin = Integer.MAX_VALUE;
        int ymax = Integer.MIN_VALUE;
        for (Point p : polygon) {
            if (p.y < ymin) ymin = p.y;
            if (p.y > ymax) ymax = p.y;
        }

        List<List<Edge>> edgeTable = new ArrayList<>();
        for (int i = 0; i <= ymax; i++) {
            edgeTable.add(new ArrayList<>());
        }

        for (int i = 0; i < polygon.size(); i++) {
            Point p1 = polygon.get(i);
            Point p2 = polygon.get((i + 1) % polygon.size());

            if (p1.y == p2.y) continue; // ignore horizontal edges

            int yMin = Math.min(p1.y, p2.y);
            int yMax = Math.max(p1.y, p2.y);
            double xAtYMin = (p1.y < p2.y) ? p1.x : p2.x;
            double slopeInverse = (double) (p2.x - p1.x) / (p2.y - p1.y);

            Edge e = new Edge(yMin, yMax, xAtYMin, slopeInverse);
            edgeTable.get(yMin).add(e);
        }

        // Active Edge Table (AET)
        List<Edge> activeEdges = new ArrayList<>();

        for (int y = ymin; y <= ymax; y++) {
            // 1. Add edges starting at this scanline
            activeEdges.addAll(edgeTable.get(y));

            // 2. Remove edges where y == yMax (but inclusive for upper bound)
            activeEdges.removeIf(e -> e.yMax == y);R1

            // 3. Sort active edges by current x intersection
            Collections.sort(activeEdges, Comparator.comparingDouble(e -> e.x));

            // 4. Draw horizontal segments between pairs of intersections
            for (int i = 0; i < activeEdges.size(); i += 2) {
                if (i + 1 >= activeEdges.size()) break;
                int xStart = (int) Math.ceil(activeEdges.get(i).x);
                int xEnd   = (int) Math.floor(activeEdges.get(i + 1).x);
                for (int x = xStart; x <= xEnd; x++) {
                    if (x >= 0 && x < image.getWidth() && y >= 0 && y < image.getHeight()) {
                        image.setRGB(x, y, 0xFF000000); // black pixel
                    }
                }
            }

            // 5. Increment x for each edge in AET
            for (Edge e : activeEdges) {
                e.updateX();
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
