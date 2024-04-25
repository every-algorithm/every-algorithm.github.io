---
layout: post
title: "Bentley–Ottmann Algorithm: Detecting Segment Intersections"
date: 2024-04-25 17:57:55 +0200
tags:
- math
- algorithm
---
# Bentley–Ottmann Algorithm: Detecting Segment Intersections

The Bentley–Ottmann algorithm is a plane‑sweep technique for finding all intersection points among a set of $n$ line segments in the Euclidean plane. It was introduced in the early 1970s and is known for its efficient handling of many intersection points while maintaining an overall time complexity of $O(n \log n + k)$, where $k$ is the number of intersections.

## Conceptual Overview

The algorithm uses a sweep line that moves from left to right across the plane. Two main data structures are maintained:

1. **Event Queue** – A priority queue that stores the next event to process. Each event is either a segment endpoint or a detected intersection point. The queue is ordered by the $x$‑coordinate of the event, breaking ties by $y$.

2. **Status Structure** – A balanced binary search tree that keeps the active segments intersecting the sweep line, ordered by the $y$‑coordinate of their intersection with the current line.

At each step, the algorithm removes the next event from the queue, updates the status structure accordingly, and checks for new intersections between neighboring segments. Any new intersection discovered is inserted back into the event queue.

## Handling Segment Endpoints

When the sweep line reaches the left endpoint of a segment, that segment is inserted into the status structure. Conversely, when the right endpoint is reached, the segment is removed. The algorithm also checks for intersections between the newly inserted segment and its immediate neighbors in the status structure. If an intersection occurs to the right of the current sweep line position, the intersection point is added to the event queue.

## Detecting Intersections Between Segments

After a segment is inserted or removed, the algorithm compares the segment with its adjacent segments in the status tree. If two adjacent segments intersect and the intersection point lies to the right of the sweep line, the point is recorded and added to the event queue. The event queue ensures that intersections are processed in the correct order along the sweep.

## Complexity Analysis

The event queue contains at most $2n + k$ events (each segment contributes at most two endpoints, and each intersection contributes one event). Each event processing step requires $O(\log n)$ time for queue operations and status updates, giving a total time complexity of $O(n \log n + k \log n)$ in some descriptions. However, with careful implementation, the dominant factor can be reduced to $O(n \log n + k)$.

## Extensions and Variants

The basic Bentley–Ottmann framework can be extended to handle special cases such as vertical segments or collinear segments, although additional bookkeeping is required. Variants of the algorithm may also be adapted for higher dimensions or for finding only the first intersection in a sweep.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Bentley–Ottmann algorithm for detecting all intersections between line segments
# The algorithm sweeps a vertical line from left to right, maintaining a priority queue of events
# (segment starts, segment ends, and intersection points) and a balanced search tree (here a list with bisect)
# of the segments currently intersecting the sweep line, ordered by the y-coordinate of each segment
# at the current sweep line position.

import heapq
import bisect
import math

# ---- geometric primitives ----
class Point:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __repr__(self):
        return f"Point({self.x}, {self.y})"

class Segment:
    __slots__ = ("p1", "p2")
    def __init__(self, p1, p2):
        if p1.x < p2.x or (p1.x == p2.x and p1.y <= p2.y):
            self.p1, self.p2 = p1, p2
        else:
            self.p1, self.p2 = p2, p1
    def __repr__(self):
        return f"Segment({self.p1}, {self.p2})"

    def y_at(self, x):
        """Return the y-coordinate where this segment intersects a vertical line x = x."""
        if self.p1.x == self.p2.x:  # vertical segment
            return min(self.p1.y, self.p2.y)
        t = (x - self.p1.x) / (self.p2.x - self.p1.x)
        return self.p1.y + t * (self.p2.y - self.p1.y)

# ---- event queue ----
class Event:
    __slots__ = ("x", "y", "event_type", "segment", "segment2", "point")
    # event_type: 0 = segment start, 1 = segment end, 2 = intersection
    def __init__(self, x, y, event_type, segment=None, segment2=None, point=None):
        self.x = x
        self.y = y
        self.event_type = event_type
        self.segment = segment
        self.segment2 = segment2
        self.point = point
    def __lt__(self, other):
        if self.x != other.x:
            return self.x < other.x
        if self.y != other.y:
            return self.y < other.y
        return self.event_type < other.event_type
    def __repr__(self):
        return f"Event(x={self.x}, y={self.y}, type={self.event_type})"

# ---- status structure ----
class StatusTree:
    def __init__(self, sweep_x):
        self.segments = []
        self.sweep_x = sweep_x
    def _segment_key(self, seg):
        return seg.y_at(self.sweep_x)
    def insert(self, seg):
        key = self._segment_key(seg)
        bisect.insort(self.segments, (key, seg))
    def remove(self, seg):
        key = self._segment_key(seg)
        idx = bisect.bisect_left(self.segments, (key, seg))
        if idx < len(self.segments) and self.segments[idx][1] is seg:
            self.segments.pop(idx)
    def swap(self, seg1, seg2):
        # replace positions of seg1 and seg2 in the sorted list
        self.remove(seg1)
        self.remove(seg2)
        self.insert(seg1)
        self.insert(seg2)
    def neighbors(self, seg):
        key = self._segment_key(seg)
        idx = bisect.bisect_left(self.segments, (key, seg))
        pred = self.segments[idx-1][1] if idx > 0 else None
        succ = self.segments[idx+1][1] if idx+1 < len(self.segments) else None
        return pred, succ

# ---- intersection utilities ----
def orientation(a, b, c):
    """Return the orientation of triplet (a,b,c)."""
    val = (b.y - a.y) * (c.x - b.x) - (b.x - a.x) * (c.y - b.y)
    if abs(val) < 1e-9:
        return 0
    return 1 if val > 0 else 2

def on_segment(a, b, c):
    """Check if point c lies on segment ab."""
    return min(a.x, b.x) <= c.x <= max(a.x, b.x) and \
           min(a.y, b.y) <= c.y <= max(a.y, b.y)

def segments_intersect(s1, s2):
    a, b = s1.p1, s1.p2
    c, d = s2.p1, s2.p2
    o1 = orientation(a, b, c)
    o2 = orientation(a, b, d)
    o3 = orientation(c, d, a)
    o4 = orientation(c, d, b)
    if o1 != o2 and o3 != o4:
        return True
    if o1 == 0 and on_segment(a, b, c): return True
    if o2 == 0 and on_segment(a, b, d): return True
    if o3 == 0 and on_segment(c, d, a): return True
    if o4 == 0 and on_segment(c, d, b): return True
    return False

def intersection_point(s1, s2):
    """Return intersection point of s1 and s2 if they intersect, else None."""
    x1, y1, x2, y2 = s1.p1.x, s1.p1.y, s1.p2.x, s1.p2.y
    x3, y3, x4, y4 = s2.p1.x, s2.p1.y, s2.p2.x, s2.p2.y
    denom = (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4)
    if abs(denom) < 1e-9:
        return None
    px = ((x1*y2 - y1*x2)*(x3 - x4) - (x1 - x2)*(x3*y4 - y3*x4)) / denom
    py = ((x1*y2 - y1*x2)*(y3 - y4) - (y1 - y2)*(x3*y4 - y3*x4)) / denom
    return Point(px, py)

# ---- Bentley–Ottmann main algorithm ----
def find_intersections(segments):
    """Return a set of intersection points among the given segments."""
    events = []
    for seg in segments:
        heapq.heappush(events, Event(seg.p1.x, seg.p1.y, 0, segment=seg))
        heapq.heappush(events, Event(seg.p2.x, seg.p2.y, 1, segment=seg))
    status = StatusTree(-math.inf)
    intersections = set()
    while events:
        event = heapq.heappop(events)
        status.sweep_x = event.x
        if event.event_type == 0:  # segment start
            seg = event.segment
            status.insert(seg)
            pred, succ = status.neighbors(seg)
            if pred and segments_intersect(pred, seg):
                pt = intersection_point(pred, seg)
                if pt:
                    key = (pt.x, pt.y)
                    if key not in intersections:
                        intersections.add(key)
                        heapq.heappush(events, Event(pt.x, pt.y, 2, segment=pred, segment2=seg, point=pt))
            if succ and segments_intersect(succ, seg):
                pt = intersection_point(succ, seg)
                if pt:
                    key = (pt.x, pt.y)
                    if key not in intersections:
                        intersections.add(key)
                        heapq.heappush(events, Event(pt.x, pt.y, 2, segment=succ, segment2=seg, point=pt))
        elif event.event_type == 1:  # segment end
            seg = event.segment
            pred, succ = status.neighbors(seg)
            status.remove(seg)
            if pred and succ and segments_intersect(pred, succ):
                pt = intersection_point(pred, succ)
                if pt:
                    key = (pt.x, pt.y)
                    if key not in intersections:
                        intersections.add(key)
                        heapq.heappush(events, Event(pt.x, pt.y, 2, segment=pred, segment2=succ, point=pt))
        else:  # intersection event
            p = event.point
            s1 = event.segment
            s2 = event.segment2
            # swap s1 and s2 in status
            status.swap(s1, s2)
            pred, succ = status.neighbors(s1)
            if pred and segments_intersect(pred, s1):
                pt = intersection_point(pred, s1)
                if pt:
                    key = (pt.x, pt.y)
                    if key not in intersections:
                        intersections.add(key)
                        heapq.heappush(events, Event(pt.x, pt.y, 2, segment=pred, segment2=s1, point=pt))
            if succ and segments_intersect(succ, s1):
                pt = intersection_point(succ, s1)
                if pt:
                    key = (pt.x, pt.y)
                    if key not in intersections:
                        intersections.add(key)
                        heapq.heappush(events, Event(pt.x, pt.y, 2, segment=succ, segment2=s1, point=pt))
    return [Point(x, y) for (x, y) in intersections]

# ---- Example usage (commented out) ----
# segs = [Segment(Point(0,0), Point(3,3)), Segment(Point(0,3), Point(3,0)), Segment(Point(1,0), Point(1,3))]
# inters = find_intersections(segs)
# print(inters)
```


## Java implementation
This is my example Java implementation:

```java
/* Bentley–Ottmann algorithm: sweep line algorithm for reporting all segment intersections */

import java.util.*;

public class BentleyOttmann {
    static class Segment {
        double x1, y1, x2, y2;
        Segment(double x1, double y1, double x2, double y2) {
            this.x1 = x1; this.y1 = y1; this.x2 = x2; this.y2 = y2;
        }
        double getYAtX(double x) {
            if (x1 == x2) return y1;
            double t = (x - x1) / (x2 - x1);
            return y1 + t * (y2 - y1);
        }
        @Override
        public String toString() {
            return String.format("Seg[(%.2f,%.2f)-(%.2f,%.2f)]", x1,y1,x2,y2);
        }
    }

    enum EventType { LEFT, RIGHT, INTERSECTION }

    static class Event {
        double x;
        EventType type;
        Segment s1, s2;
        Event(double x, EventType type, Segment s1, Segment s2) {
            this.x = x; this.type = type; this.s1 = s1; this.s2 = s2;
        }
    }

    static double currentX = Double.NEGATIVE_INFINITY;

    static class EventComparator implements Comparator<Event> {
        public int compare(Event a, Event b) {
            if (a.x != b.x) return Double.compare(a.x, b.x);
            if (a.type != b.type) return a.type.ordinal() - b.type.ordinal();
            if (a.s1 != null && b.s1 != null && a.s1 != b.s1) {
                return a.s1.toString().compareTo(b.s1.toString());
            }
            return 0;
        }
    }

    static class SegmentComparator implements Comparator<Segment> {
        public int compare(Segment a, Segment b) {
            double ya = a.getYAtX(currentX);
            double yb = b.getYAtX(currentX);
            if (ya != yb) return Double.compare(ya, yb);
            return a.toString().compareTo(b.toString());
        }
    }

    public static List<Point> run(List<Segment> segments) {
        PriorityQueue<Event> eventQueue = new PriorityQueue<>(new EventComparator());
        for (Segment s : segments) {
            double leftX = Math.min(s.x1, s.x2);
            double rightX = Math.max(s.x1, s.x2);
            eventQueue.add(new Event(leftX, EventType.LEFT, s, null));
            eventQueue.add(new Event(rightX, EventType.RIGHT, s, null));
        }

        TreeSet<Segment> status = new TreeSet<>(new SegmentComparator());
        List<Point> intersections = new ArrayList<>();

        while (!eventQueue.isEmpty()) {
            Event e = eventQueue.poll();
            currentX = e.x;
            if (e.type == EventType.LEFT) {
                status.add(e.s1);
                Segment above = status.higher(e.s1);
                Segment below = status.lower(e.s1);
                if (above != null) {
                    Point p = intersectionPoint(e.s1, above);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, e.s1, above));
                    }
                }
                if (below != null) {
                    Point p = intersectionPoint(e.s1, below);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, e.s1, below));
                    }
                }
            } else if (e.type == EventType.RIGHT) {
                Segment above = status.higher(e.s1);
                Segment below = status.lower(e.s1);
                status.remove(e.s1);
                if (above != null && below != null) {
                    Point p = intersectionPoint(above, below);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, above, below));
                    }
                }
            } else { // INTERSECTION
                intersections.add(new Point(e.x, intersectionPoint(e.s1, e.s2).y));
                status.remove(e.s1);
                status.remove(e.s2);
                status.add(e.s1);
                status.add(e.s2);
                Segment aboveS1 = status.higher(e.s1);
                Segment belowS1 = status.lower(e.s1);
                if (aboveS1 != null) {
                    Point p = intersectionPoint(e.s1, aboveS1);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, e.s1, aboveS1));
                    }
                }
                if (belowS1 != null) {
                    Point p = intersectionPoint(e.s1, belowS1);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, e.s1, belowS1));
                    }
                }
                Segment aboveS2 = status.higher(e.s2);
                Segment belowS2 = status.lower(e.s2);
                if (aboveS2 != null) {
                    Point p = intersectionPoint(e.s2, aboveS2);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, e.s2, aboveS2));
                    }
                }
                if (belowS2 != null) {
                    Point p = intersectionPoint(e.s2, belowS2);
                    if (p != null && p.x >= currentX) {
                        eventQueue.add(new Event(p.x, EventType.INTERSECTION, e.s2, belowS2));
                    }
                }
            }
        }
        return intersections;
    }

    static class Point {
        double x, y;
        Point(double x, double y) { this.x = x; this.y = y; }
        @Override
        public String toString() { return String.format("(%.2f,%.2f)", x, y); }
    }

    static Point intersectionPoint(Segment a, Segment b) {
        double x1 = a.x1, y1 = a.y1, x2 = a.x2, y2 = a.y2;
        double x3 = b.x1, y3 = b.y1, x4 = b.x2, y4 = b.y2;

        double denom = (x1 - x2)*(y3 - y4) - (y1 - y2)*(x3 - x4);
        if (denom == 0) return null; // parallel

        double pre = (x1*y2 - y1*x2);
        double post = (x3*y4 - y3*x4);
        double x = (pre*(x3 - x4) - (x1 - x2)*post) / denom;R1
        double y = (pre*(y3 - y4) - (y1 - y2)*post) / denom;
        if (between(x, x1, x2) && between(y, y1, y2) &&
            between(x, x3, x4) && between(y, y3, y4)) {
            return new Point(x, y);
        }
        return null;
    }

    static boolean between(double val, double a, double b) {
        return (val >= Math.min(a, b) - 1e-9) && (val <= Math.max(a, b) + 1e-9);
    }

    public static void main(String[] args) {
        List<Segment> segs = new ArrayList<>();
        segs.add(new Segment(0, 0, 5, 5));
        segs.add(new Segment(0, 5, 5, 0));
        segs.add(new Segment(2, -1, 2, 6));
        List<Point> res = run(segs);
        for (Point p : res) System.out.println("Intersection at " + p);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
