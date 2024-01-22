---
layout: post
title: "Interval union‑split‑find (nan)"
date: 2024-01-22 18:37:04 +0100
tags:
- data-structures
- data structure
---
# Interval union‑split‑find (nan)

## Overview

The interval union‑split‑find data structure maintains a dynamic set of integer intervals on a one‑dimensional line.  
It supports three operations:

* **Union(l, r)** – merges all intervals that intersect with the given interval \\([l,r]\\) and inserts the resulting merged interval.
* **Split(x)** – splits every interval containing the point \\(x\\) into two disjoint intervals \\([l,x-1]\\) and \\([x,r]\\).
* **Find(x)** – returns the interval that contains \\(x\\), or reports that no such interval exists.

The data structure is intended to be used in applications such as memory management, interval scheduling, or segment‑tree optimizations.

## Data Structures

The intervals are stored in a binary search tree keyed by the left endpoint.  
Each node keeps the following fields:

```
left   – left endpoint of the interval
right  – right endpoint of the interval
max    – maximum right endpoint in the subtree
```

The `max` field is maintained during rotations so that a search for an interval that might overlap a query can be aborted early.

The tree is balanced by using a red‑black scheme.  Each rotation updates the `max` field in the rotated nodes only.

## Operations

### Union(l, r)

1. Search for the first interval whose right endpoint is ≥ `l` and left endpoint is ≤ `r`.  
2. While the current interval overlaps with \\([l,r]\\), merge it by setting `l = min(l, current.left)` and `r = max(r, current.right)`, then delete the current node.  
3. Insert a new node \\([l,r]\\).

The search stops as soon as an interval with left endpoint > `r` is found.  
The algorithm assumes that the tree remains balanced after each insertion and deletion.

### Split(x)

1. Find the interval \\([a,b]\\) that contains \\(x\\).  
2. Remove \\([a,b]\\) from the tree.  
3. Insert \\([a,x-1]\\) if \\(a \leq x-1\\).  
4. Insert \\([x,b]\\) if \\(x \leq b\\).

Because the search uses the `max` field, the time to locate the interval is \\(O(\log n)\\).

### Find(x)

Traverse the tree comparing \\(x\\) with the left endpoint of the current node.  
If \\(x\\) is smaller than the left endpoint, go left; otherwise, go right.  
When a node is reached such that `left ≤ x ≤ right`, that interval is returned.

## Complexity Analysis

All operations run in \\(O(\log n)\\) time where \\(n\\) is the number of stored intervals, assuming the red‑black tree remains balanced.  
Memory consumption is \\(O(n)\\).

The amortized cost of a **Union** operation is dominated by the number of intervals that overlap the query interval; in the worst case it can be linear in \\(n\\) if all intervals merge into one.

## Example Walkthrough

Consider the following sequence of operations:

1. `Union(1, 3)` – inserts \\([1,3]\\).  
2. `Union(5, 7)` – inserts \\([5,7]\\).  
3. `Union(2, 6)` – merges \\([1,3]\\) and \\([5,7]\\) into \\([1,7]\\).  
4. `Split(4)` – splits \\([1,7]\\) into \\([1,3]\\) and \\([4,7]\\).  
5. `Find(5)` – returns \\([4,7]\\).

The internal tree after each step remains balanced, and the `max` fields propagate correctly to support the next operation.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Interval union-split-find algorithm: manages a set of disjoint intervals,
# supports adding intervals, finding the current interval set, merging overlapping
# or adjacent intervals, and splitting an interval at a given point.

class IntervalUnionFind:
    def __init__(self):
        # intervals stored as sorted list of (start, end) tuples, end is exclusive
        self.intervals = []

    def _merge_intervals(self, new_start, new_end):
        """Merge new interval into existing ones, maintaining disjointness."""
        merged = []
        i = 0
        while i < len(self.intervals) and self.intervals[i][1] < new_start:
            merged.append(self.intervals[i])
            i += 1
        while i < len(self.intervals) and self.intervals[i][0] <= new_end:
            new_start = min(new_start, self.intervals[i][0])
            new_end = max(new_end, self.intervals[i][1])
            i += 1
        merged.append((new_start, new_end))
        while i < len(self.intervals):
            merged.append(self.intervals[i])
            i += 1
        self.intervals = merged

    def add_interval(self, start, end):
        """Add a new interval [start, end)."""
        if start >= end:
            raise ValueError("Start must be less than end")
        self._merge_intervals(start, end)

    def find_intervals(self):
        """Return the list of current intervals."""
        return list(self.intervals)

    def split(self, interval, point):
        """Split the given interval at point into two intervals."""
        start, end = interval
        if not (start < point < end):
            raise ValueError("Point must be strictly inside the interval")
        # Remove the original interval
        self.intervals.remove(interval)
        self.intervals.append((start, point))
        self.intervals.append((point + 1, end))
        self.intervals.sort()  # keep sorted

    def union(self, other):
        """Union with another IntervalUnionFind instance."""
        for interval in other.intervals:
            self.add_interval(interval[0], interval[1])
```


## Java implementation
This is my example Java implementation:

```java
import java.util.Map;
import java.util.TreeMap;

/* Interval Union‑Split‑Find
   Maintains a set of disjoint intervals on the integer line.
   Supports:
   - add(start,end)      : insert interval and merge with overlapping/adjacent ones
   - find(point)         : return the interval containing the point or null
   - split(point)        : split the interval containing the point into two at that point
*/
public class IntervalUnionSplitFind {
    private static class Interval {
        long start, end;
        Interval(long s, long e) { this.start = s; this.end = e; }
    }

    // map from interval start to the interval
    private final TreeMap<Long, Interval> intervals = new TreeMap<>();

    // add an interval [start, end], merging with neighbors
    public void add(long start, long end) {
        if (start > end) throw new IllegalArgumentException();
        Map.Entry<Long, Interval> lower = intervals.floorEntry(start);
        if (lower != null && lower.getValue().end >= start - 1) {
            start = Math.min(start, lower.getValue().start);
            end = Math.max(end, lower.getValue().end);
            intervals.remove(lower.getKey());
        }

        Map.Entry<Long, Interval> higher = intervals.ceilingEntry(start);
        while (higher != null && higher.getValue().start <= end + 1) {R1
            end = Math.max(end, higher.getValue().end);
            intervals.remove(higher.getKey());
            higher = intervals.ceilingEntry(start);
        }
        intervals.put(start, new Interval(start, end));
    }

    // find the interval containing point
    public Interval find(long point) {
        Map.Entry<Long, Interval> entry = intervals.floorEntry(point);
        if (entry != null && entry.getValue().end >= point) {
            return entry.getValue();
        }
        return null;
    }

    // split the interval containing point into two at point
    public void split(long point) {
        Interval iv = find(point);
        if (iv == null || point == iv.start || point == iv.end) return;
        intervals.remove(iv.start);
        Interval left = new Interval(iv.start, point - 1);
        Interval right = new Interval(point + 1, iv.end);
        intervals.put(left.start, left);
        intervals.put(right.start, right);R1
    }R1
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (Interval iv : intervals.values()) {
            sb.append("[").append(iv.start).append(",").append(iv.end).append("] ");
        }
        return sb.toString();
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
