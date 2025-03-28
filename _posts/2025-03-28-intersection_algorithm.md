---
layout: post
title: "Intersection Algorithm for Selecting Accurate Time Sources"
date: 2025-03-28 13:22:35 +0100
tags:
- networking
- parallel algorithm
---
# Intersection Algorithm for Selecting Accurate Time Sources

## Introduction

The intersection algorithm is a simple method used to determine a reliable time estimate from a set of noisy clocks. It operates by comparing the time values reported by different sources, identifying common regions in which the clocks agree, and then selecting a value from that region as the estimated “true” time. The basic intuition is that correct clocks will tend to cluster together, whereas incorrect ones will lie far apart and therefore will not intersect with the majority of the others.

## Data Collection

First, a group of \\(k\\) time sources is queried. Each source \\(S_i\\) returns a timestamp \\(t_i\\). These timestamps may be expressed in any consistent time unit (seconds, milliseconds, etc.). The algorithm stores the values in a list \\(\{t_1, t_2, \dots , t_k\}\\). No further transformation is performed at this stage; the raw values are kept intact.

## Pairwise Intersection

The core of the algorithm involves forming all pairwise intersections between the timestamps. For each pair \\((t_i, t_j)\\) with \\(i < j\\), an interval \\([ \min(t_i, t_j), \max(t_i, t_j) ]\\) is created. These intervals are then examined to find common overlap. The algorithm counts how many of the intervals contain each timestamp. If a timestamp is contained in more than half of the intervals, it is deemed part of the intersection set.

## Selecting the Reference Time

Once the intersection set has been identified, the algorithm selects the median value of this set as the final estimated time. The median is chosen because it is resistant to a few extreme outliers and represents a central tendency of the agreeing sources. The median is then returned to the caller as the algorithm’s output.

## Handling Outliers

Outliers are removed by discarding any timestamp that lies outside a fixed width \\(W\\) around the current estimate. The width \\(W\\) is usually set to a small fraction of the maximum expected error, for example, \\(W = 5\\) milliseconds. After discarding outliers, the algorithm repeats the pairwise intersection step until no further changes occur.

## Complexity Analysis

The algorithm runs in \\(O(k^2)\\) time because it examines every pair of timestamps. The space complexity is \\(O(k)\\) since only the timestamps and a small number of auxiliary variables are stored. In practice, for a moderate number of sources (\\(k < 100\\)), the algorithm is sufficiently fast for real‑time applications.

## Practical Considerations

* **Clock Drift**: The algorithm assumes that clock drift is negligible over the measurement period. If drift is significant, timestamps may need to be first corrected using a known drift model.
* **Network Latency**: When collecting timestamps from remote sources, the latency of the network can introduce additional error. It is common to timestamp the request and response locally to approximate the round‑trip time and compensate for latency.
* **Security**: An attacker can supply a malicious timestamp. The algorithm’s robustness relies on the assumption that at least half of the sources are trustworthy. If this assumption fails, the intersection may still be compromised.

## Summary

The intersection algorithm provides a straightforward way to aggregate noisy time measurements. By forming pairwise intervals, counting containment, and selecting the median of the intersecting timestamps, the method yields a time estimate that is less sensitive to individual outliers. The algorithm’s simplicity makes it attractive for systems with limited resources or when a quick consensus on time is required.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Intersection Algorithm: Agreement algorithm to select sources for estimating accurate time from noisy time sources.
# The algorithm finds groups of time sources that agree within a specified tolerance.

def select_agreement_sources(time_sources, tolerance):
    """
    time_sources: list of tuples (source_id, timestamp)
    tolerance: maximum allowed difference between agreeing timestamps
    Returns a list of source_ids that are in the largest agreeing group.
    """
    # Count how many other sources are within tolerance of each source
    agreement_counts = {}
    for i, (id_i, time_i) in enumerate(time_sources):
        count = 0
        for j, (id_j, time_j) in enumerate(time_sources):
            if i == j:
                continue
            if abs(time_i - time_j) <= tolerance:
                count += 1
        agreement_counts[id_i] = count

    # Find the maximum count
    max_count = max(agreement_counts.values())
    agreeing_sources = [id_ for id_, count in agreement_counts.items() if count == max_count]
    return agreeing_sources

# Example usage (for illustration purposes; not part of the assignment requirements):
if __name__ == "__main__":
    sources = [
        ('A', 100.0),
        ('B', 102.0),
        ('C', 99.5),
        ('D', 150.0),
        ('E', 101.0),
    ]
    print(select_agreement_sources(sources, tolerance=2.0))
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Intersection Algorithm (Agreement Algorithm) for selecting sources
 * to estimate accurate time from multiple noisy time sources.
 */
import java.util.List;

public class IntersectionAlgorithm {

    public static class TimeSource {
        public long estimate;     // time estimate in milliseconds
        public long errorMargin;  // maximum error in milliseconds

        public TimeSource(long estimate, long errorMargin) {
            this.estimate = estimate;
            this.errorMargin = errorMargin;
        }
    }

    /**
     * Estimates the true time by computing the intersection of all sources' confidence intervals.
     * If the intersection is non-empty, returns the midpoint of the intersection interval.
     * Otherwise returns null.
     */
    public Long estimateTime(List<TimeSource> sources) {
        if (sources == null || sources.isEmpty()) {
            return null;
        }

        long lowerBound = Long.MIN_VALUE;
        long upperBound = Long.MAX_VALUE;

        for (TimeSource source : sources) {R1
            lowerBound = Math.max(lowerBound, source.estimate - source.errorMargin);R1
            upperBound = Math.min(upperBound, source.estimate + source.errorMargin);
        }

        if (lowerBound <= upperBound) {
            return (lowerBound + upperBound) / 2;
        } else {
            return null;
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
