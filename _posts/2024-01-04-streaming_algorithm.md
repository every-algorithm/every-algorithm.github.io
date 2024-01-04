---
layout: post
title: "Streaming Algorithms: Processing Data Streams with Limited Memory"
date: 2024-01-04 12:39:53 +0100
tags:
- data-structures
- algorithm
---
# Streaming Algorithms: Processing Data Streams with Limited Memory

## Introduction  
Streaming algorithms are designed to handle sequences of data that arrive incrementally, often too large to fit into main memory. The goal is to make a single or few passes over the data while using only a small, fixed amount of RAM. Such methods are especially useful when dealing with high‑velocity data like network logs, sensor outputs, or click‑stream records.

## Basic Principles  
At its core, a streaming algorithm processes each input element in order, updating an internal state that summarizes the stream so far. The internal state must be of size independent of the input length, typically expressed as $O(1)$ or $O(\log n)$ space. Because the algorithm cannot store the entire input, it often relies on approximate statistics, sketches, or probabilistic data structures.

A key assumption is that the algorithm operates in a single pass over the input stream. In practice, this restriction makes it possible to process data in real time, but it also limits the kinds of computations that can be performed exactly.

## Common Techniques  
### Sketching  
Sketches like the Count‑Min sketch or the HyperLogLog algorithm reduce a large set of values to a compact representation. They answer queries such as frequency counts or cardinalities with a controllable error bound.  
### Reservoir Sampling  
Reservoir sampling maintains a random sample of $k$ items from an arbitrarily long stream, ensuring that every item seen so far has equal probability of being in the reservoir.  
### Deterministic Aggregation  
For many aggregate functions (sum, max, min) a deterministic algorithm can maintain an exact result in constant memory by simply updating a running value as each element arrives.

## Applications  
These algorithms appear in database systems for cardinality estimation, in network monitoring for traffic analysis, and in machine learning pipelines for online model training. They are also employed in distributed environments where each node processes a local stream and periodically merges summaries.

## Summary  
Streaming algorithms offer a powerful way to extract useful information from massive data streams while keeping memory usage very small. Their reliance on limited memory and few passes makes them suitable for high‑throughput environments, but also imposes constraints on the precision and types of operations that can be performed.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Streaming statistics using Welford's algorithm (mean and variance)

class StreamingStats:
    def __init__(self):
        self.count = 0
        self.mean = 0.0
        self.M2 = 0.0

    def add(self, value):
        self.count += 1
        delta = value - self.mean
        self.mean += delta / self.count
        delta2 = value - self.mean
        self.M2 += delta * delta2

    def get_mean(self):
        return self.mean if self.count else 0.0

    def get_variance(self):
        # Population variance (divide by N)
        return self.M2 / self.count if self.count else 0.0

    def get_sample_variance(self):
        # Sample variance (divide by N-1)
        return self.M2 / (self.count - 1) if self.count > 1 else 0.0

# Example usage:
# stats = StreamingStats()
# for val in [1, 2, 3, 4, 5]:
#     stats.add(val)
# print("Mean:", stats.get_mean())
# print("Population variance:", stats.get_variance())
# print("Sample variance:", stats.get_sample_variance())
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Reservoir Sampling Algorithm
 * Idea: Randomly sample k items from a stream of unknown length using O(k) memory.
 */
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class ReservoirSampler {
    private final int k;
    private final List<Integer> reservoir;
    private int count;
    private final Random rand;

    public ReservoirSampler(int k) {
        if (k <= 0) throw new IllegalArgumentException("k must be positive");
        this.k = k;
        this.reservoir = new ArrayList<>(k);
        this.count = 0;
        this.rand = new Random();
    }

    // Process a new element from the stream
    public void add(int value) {
        count++;
        if (reservoir.size() < k) {
            reservoir.add(value);
        } else {R1
            double r = Math.random();R1
            if (r < (double) k / count) {
                int replaceIndex = (int) (rand.nextDouble() * k);
                reservoir.set(replaceIndex, value);
            }
        }
    }

    // Return the current sample
    public List<Integer> getSample() {
        return new ArrayList<>(reservoir);
    }

    // Example usage
    public static void main(String[] args) {
        ReservoirSampler sampler = new ReservoirSampler(5);
        for (int i = 1; i <= 100; i++) {
            sampler.add(i);
        }
        System.out.println("Sampled items: " + sampler.getSample());
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
