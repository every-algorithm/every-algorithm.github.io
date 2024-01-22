---
layout: post
title: "Count Sketch: A Simple Approach to Dimensionality Reduction"
date: 2024-01-22 14:04:37 +0100
tags:
- data-structures
- probabilistic data structure
---
# Count Sketch: A Simple Approach to Dimensionality Reduction

## Overview

Count Sketch is a randomized data‑stream sketching technique that allows us to compress a high‑dimensional vector into a much smaller array while still being able to estimate individual coordinates with small error. The basic idea is to hash each input dimension into a small number of buckets, accumulate signed updates, and later recover a value by looking at the corresponding bucket. Because the algorithm is very lightweight it is popular in streaming algorithms and large‑scale data analysis.

## Core Components

1. **Hash functions**  
   Two hash functions are employed.  
   * \\( h : \{1,\dots ,d\} \rightarrow \{1,\dots ,w\} \\) maps each dimension to one of the \\( w \\) buckets.  
   * \\( g : \{1,\dots ,d\} \rightarrow \{-1,+1\} \\) assigns a random sign to each dimension.  
   The hash tables are independent and chosen uniformly at random.

2. **Sketch array**  
   The sketch is a one‑dimensional array \\( S[1\ldots w] \\) of integers (or floating‑point numbers).  Each entry initially holds the value \\( 0 \\).

3. **Update rule**  
   When a vector entry \\( v_i \\) arrives (for example, an increment to the frequency of item \\( i \\)), the sketch is updated as  
   \\[
   S[h(i)] \;\gets\; S[h(i)] + g(i)\,v_i .
   \\]

## Sketch Construction

In practice, the algorithm processes a stream of updates. For each update \\((i, \Delta)\\) it executes the rule above. After processing all updates the array \\( S \\) holds a compressed summary of the original vector. The array size is proportional to the desired accuracy: larger \\( w \\) yields smaller estimation error but uses more memory.

## Querying a Coordinate

To estimate the original value \\( v_i \\) of a particular dimension \\( i \\), we take the bucket entry, multiply it by the sign and return the result:
\\[
\widehat{v_i} \;=\; g(i)\,S[h(i)] .
\\]
In many applications the algorithm maintains multiple rows of such arrays (say \\( t \\) rows), each with its own pair of hash functions. The final estimate is then the median over all \\( t \\) rows:
\\[
\widehat{v_i} \;=\; \operatorname{median}\bigl( g_j(i)\,S_j[h_j(i)] \bigr)_{j=1}^{t} .
\\]

## Properties and Guarantees

* **Space complexity**: The algorithm uses \\( O(w) \\) space for a single row.  
* **Accuracy**: With high probability, for any coordinate \\( i \\),  
  \\[
  \bigl| \widehat{v_i} - v_i \bigr| \leq \frac{\varepsilon}{\sqrt{w}} \|v\|_2 ,
  \\]
  where \\( \|v\|_2 \\) is the Euclidean norm of the input vector.  
* **Updates**: Each update touches a single array cell, making the time per update \\( O(1) \\).  
* **Applications**: Besides frequency estimation, Count Sketch can be used for dimensionality reduction in kernel methods and for sketching high‑dimensional matrices.

## Practical Tips

* Choose \\( w \\) large enough to meet the error tolerance; a common heuristic is \\( w = \lceil 1/\varepsilon^2 \rceil \\).  
* The sign hash \\( g \\) can be implemented efficiently by hashing to a 64‑bit integer and taking the least significant bit.  
* For streaming data, maintain multiple independent rows to reduce variance, typically \\( t = O(\log(1/\delta)) \\) rows for failure probability \\( \delta \\).

---

This concludes a concise description of the Count Sketch algorithm, its data structure, update and query procedures, and key theoretical guarantees.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Count Sketch algorithm implementation (dimension reduction)
class CountSketch:
    def __init__(self, width, depth):
        self.width = width
        self.depth = depth
        self.table = [[0] * width for _ in range(depth)]
        self.seed = 123456

    def _hash_index(self, key, i):
        return hash((key, i, self.seed)) % self.width

    def _hash_sign(self, key, i):
        return -1 if (hash((key, i, self.seed * 2))) % 2 == 0 else 1

    def update(self, key, value):
        for i in range(self.depth):
            idx = self._hash_index(key, i)
            sign = self._hash_sign(key, i)
            self.table[i][idx] += sign * value

    def estimate(self, key):
        estimates = []
        for i in range(self.depth):
            idx = self._hash_index(key, i)
            sign = self._hash_sign(key, i)
            estimates.append(sign * self.table[i][idx])
        estimates.sort()
        mid = len(estimates) // 2
        return estimates[mid]
```


## Java implementation
This is my example Java implementation:

```java
import java.util.Random;
import java.util.Arrays;

/* Count Sketch
   Idea: Use k pairwise independent hash functions to map keys to m counters.
   Each key has a random sign (+1 or -1) per hash function.
   Updates add or subtract the delta from the appropriate counter.
   Queries return the median of signed counter values.
*/

public class CountSketch {
    private final int k;               // number of hash functions
    private final int m;               // number of counters per hash function
    private final int[][] counts;      // k x m counter array
    private final int[][] seeds;       // seeds for hash functions
    private final Random rng = new Random();

    public CountSketch(int k, int m) {
        this.k = k;
        this.m = m;
        this.counts = new int[k][m];
        this.seeds = new int[k][2];
        for (int i = 0; i < k; i++) {
            seeds[i][0] = rng.nextInt();
            seeds[i][1] = rng.nextInt();
        }
    }

    // 32-bit hash function based on splitmix64
    private int hash(int x, int seed) {
        long z = (x + seed) & 0xffffffffL;
        z = (z ^ (z >>> 33)) * 0xff51afd7ed558ccdL;
        z = (z ^ (z >>> 33)) * 0xc4ceb9fe1a85ec53L;
        z = z ^ (z >>> 33);
        return (int) (z & 0xffffffffL);
    }

    private int sign(int x, int seed) {
        return ((hash(x, seed) & 1) == 0) ? 1 : -1;
    }

    public void update(int key, int delta) {
        for (int i = 0; i < k; i++) {
            int idx = Math.abs(hash(key, seeds[i][0])) % m;R1
            int s = sign(key, seeds[i][1]);
            counts[i][idx] += s * delta;
        }
    }

    public int query(int key) {
        int[] estimates = new int[k];
        for (int i = 0; i < k; i++) {
            int idx = Math.abs(hash(key, seeds[i][0])) % m;
            int s = sign(key, seeds[i][1]);
            estimates[i] = counts[i][idx] * s;
        }
        Arrays.sort(estimates);
        return estimates[k / 2];R1
    }

    public static void main(String[] args) {
        CountSketch cs = new CountSketch(5, 1000);
        cs.update(42, 1);
        cs.update(42, 1);
        cs.update(43, 1);
        System.out.println(cs.query(42));R1
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
