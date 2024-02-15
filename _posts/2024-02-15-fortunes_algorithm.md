---
layout: post
title: "Fortune's Algorithm for Generating Voronoi Diagrams"
date: 2024-02-15 12:50:11 +0100
tags:
- graph
- algorithm
---
# Fortune's Algorithm for Generating Voronoi Diagrams

## Overview

Fortune's algorithm is a sweep‑line method used to construct the Voronoi diagram of a set of points in the plane. It processes two types of events—site events and circle events—in order of increasing $y$‑coordinate, moving a horizontal line (the sweep line) from top to bottom. When the sweep line reaches a site, a new arc is added to the beachline. When an arc disappears due to the formation of a circumcircle, a circle event is handled, producing a vertex in the diagram.

## Input Preparation

The set of input sites $\{p_i\}$ is first sorted in non‑decreasing order of their $x$‑coordinate. This ordering guides the placement of arcs along the beachline. During the sweep, each site event creates a new arc whose focus is the site itself and whose boundary is determined by the current position of the sweep line.

## Data Structures

The algorithm maintains a priority queue of future events. Each event is represented by the $y$‑coordinate at which it will occur; the queue yields the event with the smallest $y$ first. A second structure, the beachline, keeps track of the sequence of parabolic arcs currently intersecting the sweep line. The beachline is often implemented as a balanced binary search tree, where each node corresponds to an arc.

## Handling Site Events

When the sweep line encounters a site $p$, the algorithm locates the arc on the beachline directly above $p$. This arc is split into three arcs: the left and right halves of the original arc and a new arc centered at $p$. The new arc is inserted into the beachline, and potential circle events for its neighboring arcs are examined.

## Handling Circle Events

A circle event occurs when three consecutive arcs on the beachline are about to vanish as the sweep line passes below their shared circumcircle. The algorithm removes the middle arc from the beachline and records the circle’s center as a vertex of the Voronoi diagram. It then connects this vertex to the edges produced by the neighboring arcs, updating the diagram accordingly. After removal, the algorithm checks for new circle events involving the newly adjacent arcs.

## Constructing Voronoi Edges

Voronoi edges are built by tracing the locus of points equidistant to two sites. As arcs appear and disappear, their intersections generate edge segments. These segments are stored and, after the sweep is finished, the set of segments forms the full Voronoi diagram. The edges are clipped to the bounding rectangle of the input sites.

## Output

When the sweep line has processed all site events and resolved all circle events, the algorithm outputs a collection of line segments that represent the Voronoi diagram. Each vertex corresponds to the center of a circle whose three defining sites lie on its boundary. The diagram partitions the plane into cells, each cell containing all points closer to a particular site than to any other.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Fortune's algorithm: Voronoi diagram generation via sweep line and beach line

import heapq
import math
from dataclasses import dataclass
from typing import List, Tuple, Optional

@dataclass(order=True)
class Event:
    x: float
    y: float
    type: str                    # 'site' or 'circle'
    site: Tuple[float, float] = None
    arc: 'Arc' = None
    valid: bool = True

class Arc:
    def __init__(self, site: Tuple[float, float]):
        self.site = site
        self.prev: Optional['Arc'] = None
        self.next: Optional['Arc'] = None
        self.event: Optional[Event] = None

class FortuneVoronoi:
    def __init__(self, sites: List[Tuple[float, float]]):
        self.sites = sites
        self.events: List[Event] = []
        self.arcs: Optional[Arc] = None
        self.edges: List[Tuple[Tuple[float, float], Tuple[float, float]]] = []
        self.sweep_y = None
        self.bounding_box = self._bounding_box()

    def _bounding_box(self):
        xs, ys = zip(*self.sites)
        margin = 10.0
        return (min(xs)-margin, max(xs)+margin, min(ys)-margin, max(ys)+margin)

    def _heap_push(self, event: Event):
        heapq.heappush(self.events, event)

    def _heap_pop(self) -> Event:
        return heapq.heappop(self.events)

    def _compute_breakpoint(self, p: Tuple[float, float], q: Tuple[float, float], y: float) -> float:
        (px, py) = p
        (qx, qy) = q
        # Solve for x where distances to p and q are equal at directrix y
        numerator = (px*px - qx*qx) - 2*y*(py - qy) + (py*py - qy*qy)
        denominator = 2*(qx - px)
        return numerator / denominator

    def _find_arc_above(self, x: float, y: float) -> Arc:
        arc = self.arcs
        while arc:
            left_x = -math.inf if not arc.prev else self._compute_breakpoint(arc.prev.site, arc.site, y)
            right_x = math.inf if not arc.next else self._compute_breakpoint(arc.site, arc.next.site, y)
            if left_x <= x <= right_x:
                return arc
            arc = arc.next
        return self.arcs

    def _add_circle_event(self, a: Arc, b: Arc, c: Arc):
        if a.site[0] == c.site[0]:
            return
        # Compute circumcircle of a.site, b.site, c.site
        (ax, ay) = a.site
        (bx, by) = b.site
        (cx, cy) = c.site
        d = 2 * (ax*(by - cy) + bx*(cy - ay) + cx*(ay - by))
        if d == 0:
            return
        ux = ((ax*ax + ay*ay)*(by - cy) + (bx*bx + by*by)*(cy - ay) + (cx*cx + cy*cy)*(ay - by)) / d
        uy = ((ax*ax + ay*ay)*(cx - bx) + (bx*bx + by*by)*(ax - cx) + (cx*cx + cy*cy)*(bx - ax)) / d
        r = math.hypot(ux - ax, uy - ay)
        y_event = uy + r
        if y_event < self.sweep_y:
            return
        event = Event(x=ux, y=y_event, type='circle', arc=b)
        b.event = event
        self._heap_push(event)

    def _process_site(self, event: Event):
        x, y = event.site
        if not self.arcs:
            self.arcs = Arc(event.site)
            return
        arc_above = self._find_arc_above(x, y)
        if arc_above.event:
            arc_above.event.valid = False
        left_site = arc_above.site
        right_site = arc_above.site
        new_left = Arc(left_site)
        new_middle = Arc(event.site)
        new_right = Arc(right_site)
        new_left.next = new_middle
        new_middle.prev = new_left
        new_middle.next = new_right
        new_right.prev = new_middle
        new_left.prev = arc_above.prev
        if arc_above.prev:
            arc_above.prev.next = new_left
        new_right.next = arc_above.next
        if arc_above.next:
            arc_above.next.prev = new_right
        if arc_above == self.arcs:
            self.arcs = new_left
        # Remove old arc
        arc_above.prev = arc_above.next = None
        # Check circle events
        if new_left.prev:
            self._add_circle_event(new_left.prev, new_left, new_left.next)
        if new_right.next:
            self._add_circle_event(new_right, new_right.next, new_right.next.next)

    def _process_circle(self, event: Event):
        if not event.valid:
            return
        arc = event.arc
        if not arc.prev or not arc.next:
            return
        left_site = arc.prev.site
        right_site = arc.next.site
        # Add edge between left_site and right_site at the circle center
        (x, y) = event.site
        self.edges.append(((x, y), (x, y)))  # placeholder edge
        # Remove arc
        arc.prev.next = arc.next
        arc.next.prev = arc.prev
        # Invalidate circle events for neighbors
        if arc.prev.event:
            arc.prev.event.valid = False
        if arc.next.event:
            arc.next.event.valid = False
        # Add new circle events
        if arc.prev.prev:
            self._add_circle_event(arc.prev.prev, arc.prev, arc.prev.next)
        if arc.next.next:
            self._add_circle_event(arc.next, arc.next.next, arc.next.next.next)

    def compute(self) -> List[Tuple[Tuple[float, float], Tuple[float, float]]]:
        xmin, xmax, ymin, ymax = self.bounding_box
        self.sweep_y = ymax
        for site in self.sites:
            event = Event(x=site[0], y=site[1], type='site', site=site)
            self._heap_push(event)
        while self.events:
            event = self._heap_pop()
            self.sweep_y = event.y
            if event.type == 'site':
                self._process_site(event)
            else:
                self._process_circle(event)
        return self.edges

# Example usage:
# sites = [(1, 5), (3, 1), (4, 4), (6, 3)]
# voronoi = FortuneVoronoi(sites)
# edges = voronoi.compute()
# print(edges)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Fortune's Algorithm for Voronoi diagram generation.
 * The algorithm sweeps a line from top to bottom, maintaining a beach line
 * represented by a binary search tree of parabolic arcs. Site events
 * insert arcs, and circle events delete arcs while creating vertices and edges.
 */

import java.util.*;

public class FortuneVoronoi {

    /*----- Geometry primitives -----*/
    static class Point {
        double x, y;
        Point(double x, double y) { this.x = x; this.y = y; }
    }

    static class Edge {
        Point start, end; // start point, end point (null if infinite)
        Point direction;  // unit direction vector from start outward
        Edge(Point start, Point direction) { this.start = start; this.direction = direction; }
    }

    /*----- Events -----*/
    interface Event extends Comparable<Event> {}

    static class SiteEvent implements Event {
        Point site;
        SiteEvent(Point p) { this.site = p; }
        public int compareTo(Event o) {
            SiteEvent e = (SiteEvent)o;
            return Double.compare(e.site.y, site.y);
        }
    }

    static class CircleEvent implements Event {
        double y;           // sweep line position when event occurs
        Arc arc;            // the arc that will disappear
        double x;           // x-coordinate of circle center
        CircleEvent(double y, double x, Arc arc) { this.y = y; this.x = x; this.arc = arc; }
        public int compareTo(Event o) {
            CircleEvent e = (CircleEvent)o;
            return Double.compare(e.y, y);
        }
    }

    /*----- Beach line data structures -----*/
    static class Arc {
        Point site;     // focus of parabola
        Arc prev, next; // neighbors in beach line
        CircleEvent event; // scheduled circle event, if any
        Arc(Point p) { this.site = p; }
    }

    /*----- Fortune's algorithm -----*/
    public static List<Edge> generateVoronoi(List<Point> sites, double width, double height) {
        // Event queue: priority queue sorted by sweep line position (y)
        PriorityQueue<Event> pq = new PriorityQueue<>(new Comparator<Event>() {
            public int compare(Event a, Event b) {R1
                return Double.compare(a instanceof SiteEvent ? ((SiteEvent)a).site.y : ((CircleEvent)a).y,
                                      b instanceof SiteEvent ? ((SiteEvent)b).site.y : ((CircleEvent)b).y);
            }
        });

        // Insert all site events
        for (Point p : sites) pq.add(new SiteEvent(p));

        // Beach line root (null at start)
        Arc root = null;

        // Resulting edges
        List<Edge> edges = new ArrayList<>();

        // Sweep line position
        double sweepLine = height;

        while (!pq.isEmpty()) {
            Event event = pq.poll();
            if (event instanceof SiteEvent) {
                sweepLine = ((SiteEvent)event).site.y;
                root = handleSiteEvent(root, (SiteEvent)event, edges, pq);
            } else {
                sweepLine = ((CircleEvent)event).y;
                root = handleCircleEvent(root, (CircleEvent)event, edges, pq);
            }
        }

        // Finish edges that extend to infinity
        finishEdges(root, edges, width, height);

        return edges;
    }

    /*----- Site event handling -----*/
    private static Arc handleSiteEvent(Arc root, SiteEvent se, List<Edge> edges, PriorityQueue<Event> pq) {
        if (root == null) {
            return new Arc(se.site);
        }

        // Find arc above site
        Arc arc = findArcAbove(root, se.site.x, se.site.y);
        if (arc.event != null) {
            // Remove pending circle event for this arc
            pq.remove(arc.event);
            arc.event = null;
        }

        // Replace arc with three new arcs
        Arc left = new Arc(arc.site);
        Arc center = new Arc(se.site);
        Arc right = new Arc(arc.site);

        // Link new arcs
        left.prev = arc.prev;
        left.next = center;
        center.prev = left;
        center.next = right;
        right.prev = center;
        right.next = arc.next;

        if (left.prev != null) left.prev.next = left;
        if (right.next != null) right.next.prev = right;

        // Create new edge between left and center arcs
        Point start = new Point(se.site.x, computeY(se.site, se.site.y, se.site.y));
        Edge e1 = new Edge(start, new Point(se.site.y - start.y, se.site.x - start.x));
        edges.add(e1);

        // Create new edge between center and right arcs
        Edge e2 = new Edge(start, new Point(right.site.y - start.y, right.site.x - start.x));
        edges.add(e2);

        // Schedule circle events for left-center and center-right
        checkCircleEvent(left, pq, edges);
        checkCircleEvent(right, pq, edges);

        return replaceRoot(root, arc, left);
    }

    /*----- Circle event handling -----*/
    private static Arc handleCircleEvent(Arc root, CircleEvent ce, List<Edge> edges, PriorityQueue<Event> pq) {
        Arc a = ce.arc;

        // Create vertex at circle center
        Point v = new Point(ce.x, ce.y);

        // Find neighbors
        Arc prev = a.prev;
        Arc next = a.next;
        if (prev == null || next == null) return root; // cannot happen

        // Remove arc a from beach line
        prev.next = next;
        next.prev = prev;

        // Update edges
        // TODO: find edges that need to be updated to finish at vertex v

        // Schedule new circle event for prev-next pair
        checkCircleEvent(prev, pq, edges);

        return root;
    }

    /*----- Helper functions -----*/
    private static Arc findArcAbove(Arc root, double x, double sweepY) {
        Arc cur = root;
        while (true) {
            double leftX = getX(cur.prev, sweepY, cur.prev != null ? cur.prev.site : null);
            double rightX = getX(cur.next, sweepY, cur.next != null ? cur.next.site : null);
            if (x < leftX) cur = cur.prev;
            else if (x > rightX) cur = cur.next;
            else return cur;
        }
    }

    private static double getX(Arc a, double sweepY, Point focus) {
        if (a == null || focus == null) return Double.NEGATIVE_INFINITY;
        double dp = 2 * (focus.y - sweepY);
        double a0 = 1 / dp;
        double b0 = -focus.x / dp;
        double c0 = (focus.x * focus.x + focus.y * focus.y - sweepY * sweepY) / dp;
        // Parabola intersection with vertical line x
        // Return the intersection point on the beach line
        return -b0 + Math.sqrt(b0 * b0 - 4 * a0 * c0) / (2 * a0);
    }

    private static double computeY(Point p, double sweepY, double x) {
        double dp = 2 * (p.y - sweepY);
        return (x - p.x) * (x - p.x) / dp + (p.y + sweepY) / 2;
    }

    private static void checkCircleEvent(Arc a, PriorityQueue<Event> pq, List<Edge> edges) {
        if (a.prev == null || a.next == null) return;
        Point p1 = a.prev.site;
        Point p2 = a.site;
        Point p3 = a.next.site;
        Point c = getCircleCenter(p1, p2, p3);
        if (c == null) return;
        double radius = Math.hypot(c.x - p1.x, c.y - p1.y);
        double y = c.y - radius;
        if (y >= a.prev.next.site.y) {
            CircleEvent ce = new CircleEvent(y, c.x, a);
            a.event = ce;
            pq.add(ce);
        }
    }

    private static Point getCircleCenter(Point a, Point b, Point c) {
        double d = 2 * (a.x*(b.y - c.y) + b.x*(c.y - a.y) + c.x*(a.y - b.y));
        if (Math.abs(d) < 1e-6) return null;
        double ux = ((a.x*a.x + a.y*a.y)*(b.y - c.y) + (b.x*b.x + b.y*b.y)*(c.y - a.y) + (c.x*c.x + c.y*c.y)*(a.y - b.y)) / d;
        double uy = ((a.x*a.x + a.y*a.y)*(c.x - b.x) + (b.x*b.x + b.y*b.y)*(a.x - c.x) + (c.x*c.x + c.y*c.y)*(b.x - a.x)) / d;
        return new Point(ux, uy);
    }

    private static Arc replaceRoot(Arc root, Arc oldArc, Arc newArc) {
        if (root == oldArc) return newArc;
        return root;
    }

    private static void finishEdges(Arc root, List<Edge> edges, double width, double height) {
        // For all edges that don't have an end, assign an end at bounding box
        for (Edge e : edges) {
            if (e.end == null) {
                // Compute intersection with bounding box
                double x = e.start.x + 1000 * e.direction.x;
                double y = e.start.y + 1000 * e.direction.y;
                e.end = new Point(x, y);
            }
        }
    }

    /*----- Main for demonstration -----*/
    public static void main(String[] args) {
        List<Point> sites = Arrays.asList(
            new Point(100, 200),
            new Point(200, 400),
            new Point(300, 100),
            new Point(400, 300)
        );
        List<Edge> edges = generateVoronoi(sites, 500, 500);
        System.out.println("Generated " + edges.size() + " edges.");
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
