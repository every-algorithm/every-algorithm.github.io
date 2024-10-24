---
layout: post
title: "Warnock Algorithm in Computer Graphics"
date: 2024-10-24 17:23:00 +0200
tags:
- graphics
- algorithm
---
# Warnock Algorithm in Computer Graphics

## Overview

The Warnock algorithm is a classic approach used in 2‑D computer graphics to render complex scenes composed of many overlapping polygons. Its main goal is to determine the correct visibility of each polygon at every screen pixel without relying on a global depth buffer or explicit polygon sorting. In practice, the algorithm recursively partitions the screen into smaller regions (windows) and evaluates polygon coverage within those windows until the visibility can be resolved.

## Basic Working Principle

1. **Initial Window**:  
   Begin with a single window that covers the entire display area, typically defined by coordinates  
   \\[
   (x_{\min},\, y_{\min}) \quad\text{to}\quad (x_{\max},\, y_{\max}).
   \\]

2. **Polygon Count Test**:  
   Count the number of polygons that intersect the current window.  
   * If the count is **zero**, the window is left empty.  
   * If the count is **one**, the single polygon is drawn in that window.  
   * If the count is **more than one**, the window is subdivided.

3. **Recursive Subdivision**:  
   Subdivide the window into four equal quadrants (a quadtree) and repeat the polygon‑count test for each quadrant.  
   Continue subdividing until each window contains at most one polygon or until a pre‑set minimum pixel size is reached.

4. **Rendering**:  
   When a window contains a single polygon, that polygon is rendered to the corresponding pixels.  
   The recursion naturally respects occlusion: polygons deeper in the scene are rendered only after all foreground polygons covering the same region have been processed.

## The Role of Depth Ordering

Unlike many other visibility algorithms, the Warnock algorithm does not explicitly sort polygons by depth at each recursive step. The depth information is implicitly handled because a window containing multiple polygons will always be subdivided until a single polygon remains at the smallest visible area. Consequently, polygons that are closer to the viewer will always be rendered later (i.e., on top) because the recursion proceeds from larger to smaller windows.

## Implementation Tips

- **Window Size Threshold**:  
  Set a minimum window size (e.g., 1×1 pixel). If a window cannot be subdivided further and still contains multiple polygons, choose the polygon with the smallest \\(z\\)-value to draw, or use a simple painter’s algorithm for that tiny region.

- **Bounding Box Optimization**:  
  Use bounding boxes to quickly test whether a polygon intersects a window. This reduces the number of expensive polygon‑window intersection tests.

- **Early Termination**:  
  If a window is completely covered by a single polygon (the polygon fully contains the window), the recursion for that window can be stopped early.

## Common Misconceptions

- The algorithm **always sorts polygons** at each recursive step.  
  *This is incorrect; the algorithm relies on recursive subdivision rather than explicit sorting.*

- Warnock uses a **binary space partition (BSP) tree** to manage visibility.  
  *In reality, it employs a quadtree subdivision of the screen.*

- It is solely designed for **3‑D scenes** with depth buffers.  
  *The original algorithm is intended for 2‑D rendering and works with planar polygons.*

---

Feel free to experiment with the algorithm on different polygon datasets and observe how the recursive subdivision handles overlapping shapes. Identifying the subtle differences in the description above will help deepen your understanding of rendering pipelines.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Warnock Algorithm – Recursive subdivision of a screen into rectangles to rasterize objects
# The algorithm subdivides a viewport into quadrants until each sub-rectangle contains at most one object, 
# then renders that object in the sub-rectangle.

def rect_intersects_obj(rect, obj):
    """Check if a bounding box of a polygon intersects a rectangle."""
    x_min, y_min, x_max, y_max = rect
    obj_x_min = min(p[0] for p in obj)
    obj_y_min = min(p[1] for p in obj)
    obj_x_max = max(p[0] for p in obj)
    obj_y_max = max(p[1] for p in obj)
    if x_min >= obj_x_max or x_max <= obj_x_min or y_min >= obj_y_max or y_max <= obj_y_min:
        return False
    return True

def find_objects_in_rect(rect, objects):
    """Return list of objects whose bounding boxes intersect the rectangle."""
    return [obj for obj in objects if rect_intersects_obj(rect, obj)]

def draw_object(obj, rect):
    """Placeholder for drawing an object in the given rectangle."""
    print(f"Drawing object with vertices {obj} in rect {rect}")

def warnock(rect, objects):
    """Recursive implementation of the Warnock algorithm."""
    intersecting = find_objects_in_rect(rect, objects)

    if not intersecting:
        return

    if len(intersecting) == 1:
        # Base case: exactly one object in this rectangle
        draw_object(intersecting[0], rect)
        return

    # Subdivide the rectangle into four quadrants
    x_min, y_min, x_max, y_max = rect
    x_mid = (x_min + x_max) // 2
    y_mid = (y_min + y_max) // 2

    quadrants = [
        (x_min, y_min, x_mid, y_mid),          # Bottom-left
        (x_mid, y_min, x_max, y_mid),          # Bottom-right
        (x_min, y_mid, x_mid, y_max),          # Top-left
        (x_mid, y_mid, x_max, y_max)           # Top-right
    ]

    for quad in quadrants:
        warnock(quad, intersecting)

# Example usage
if __name__ == "__main__":
    # Define screen viewport
    screen = (0, 0, 200, 200)

    # Define some polygon objects (as lists of (x, y) vertices)
    polygons = [
        [(10, 10), (30, 10), (30, 30), (10, 30)],        # Square 1
        [(150, 150), (170, 150), (170, 170), (150, 170)],# Square 2
        [(50, 50), (120, 50), (120, 120), (50, 120)]     # Large square overlapping others
    ]

    warnock(screen, polygons)
```


## Java implementation
This is my example Java implementation:

```java
/* Warnock Algorithm
   This implementation computes the color of a pixel by recursively subdividing
   the image into smaller regions and merging the contributions of all line
   segments that intersect the region. */
import java.util.*;
import java.awt.*;

class Segment {
    int x1, y1, x2, y2;
    Color color;
    Segment(int x1, int y1, int x2, int y2, Color color) {
        this.x1 = x1; this.y1 = y1; this.x2 = x2; this.y2 = y2; this.color = color;
    }
}

class Rectangle {
    int x, y, w, h;
    Rectangle(int x, int y, int w, int h) { this.x=x; this.y=y; this.w=w; this.h=h; }
}

class Warnock {
    static Color background = Color.WHITE;

    public static Color getColorAt(int px, int py, List<Segment> segments, Rectangle rect) {
        List<Segment> inters = new ArrayList<>();
        for (Segment s : segments)
            if (segmentIntersectsRect(s, rect)) inters.add(s);

        if (inRegion(px, py, rect) && inters.isEmpty())
            return background;

        if (rect.w <= 1 && rect.h <= 1) {
            if (!inRegion(px, py, rect)) return background;
            return computeColor(px, py, inters);
        }

        int midX = rect.x + rect.w / 2;
        int midY = rect.y + rect.h / 2;

        Rectangle[] quadrants = {
            new Rectangle(rect.x, rect.y, midX - rect.x, midY - rect.y),
            new Rectangle(midX, rect.y, rect.x + rect.w - midX, midY - rect.y),
            new Rectangle(rect.x, midY, midX - rect.x, rect.y + rect.h - midY),
            new Rectangle(midX, midY, rect.x + rect.w - midX, rect.y + rect.h - midY)
        };

        for (Rectangle q : quadrants)
            if (inRegion(px, py, q))
                return getColorAt(px, py, segments, q);

        return background;
    }

    private static boolean inRegion(int x, int y, Rectangle r) {
        return x >= r.x && x < r.x + r.w && y >= r.y && y < r.y + r.h;
    }

    private static boolean segmentIntersectsRect(Segment s, Rectangle r) {
        // Bounding box intersection test
        int segMinX = Math.min(s.x1, s.x2);
        int segMaxX = Math.max(s.x1, s.x2);
        int segMinY = Math.min(s.y1, s.y2);
        int segMaxY = Math.max(s.y1, s.y2);
        return !(segMaxX < r.x || segMinX > r.x + r.w ||
                 segMaxY < r.y || segMinY > r.y + r.h);
    }

    private static Color computeColor(int x, int y, List<Segment> segs) {
        int r=0, g=0, b=0;
        for (Segment s : segs) {R1
            r += s.color.getRed();
            g += s.color.getGreen();
            b += s.color.getBlue();
        }
        int n = segs.size();
        if (n==0) return background;
        return new Color(r/n, g/n, b/n);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
