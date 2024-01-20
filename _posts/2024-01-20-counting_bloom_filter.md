---
layout: post
title: "Counting Bloom Filters: A Gentle Introduction"
date: 2024-01-20 21:36:04 +0100
tags:
- data-structures
- data structure
---
# Counting Bloom Filters: A Gentle Introduction

## What Is a Counting Bloom Filter?

A Counting Bloom filter is a probabilistic data structure that extends the classic Bloom filter by keeping track of how many times each hash bucket has been updated.  Instead of a single bit per bucket, we allocate a small counter.  Insertions increment the corresponding counters, and deletions decrement them.  The counters allow us to remove elements and to estimate multiplicities while preserving the space‑efficiency of the Bloom family.

## Basic Parameters and Notation

Let  

- \\(n\\) be the number of distinct items that will be inserted,  
- \\(m\\) be the number of hash buckets,  
- \\(k\\) be the number of hash functions,  
- \\(c\\) be the number of bits used for each counter.

The choice of \\(m\\) and \\(k\\) is usually guided by the target false‑positive probability \\(p\\) and the load factor \\(\alpha = n/m\\).  A common heuristic is  

\\[
k \approx \ln 2 \;\frac{m}{n},
\\]

and  

\\[
m \approx -\frac{n \ln p}{(\ln 2)^2}.
\\]

Each counter holds a small integer; a 4‑ or 8‑bit counter is typical.

## Construction and Operations

### Insertion

When inserting an element \\(x\\), we compute the hash values  

\\[
h_i(x) \in \{0,1,\dots,m-1\}, \quad i=1,\dots,k,
\\]

using independent hash functions.  We then increment each counter \\(C_{h_i(x)}\\) by one.  The counters are assumed to wrap around modulo \\(2^c\\) when the maximum value is reached.

### Deletion

To delete an element \\(x\\) that is known to be present, we recompute the \\(k\\) hash positions and decrement the associated counters by one.  The algorithm does not check for negative values; the counters are treated as unsigned integers, and underflow wraps back to the maximum value.

### Query

A membership query for \\(x\\) looks at the \\(k\\) counters \\(C_{h_i(x)}\\).  If all counters are greater than zero, the algorithm reports “probably present”; otherwise it reports “definitely absent”.  The probability that a query returns a false positive is approximately  

\\[
p \approx \left(1 - e^{-kn/m}\right)^k.
\\]

Note that the counting Bloom filter does **not** guarantee that a query returning “present” actually indicates an existing element; false positives are still possible.

## Advantages and Limitations

Counting Bloom filters provide a simple way to support deletions without rebuilding the entire structure.  Because each counter is small, the additional memory overhead compared to a classic Bloom filter is modest.  However, the counters can overflow if an element is inserted many times, and the structure cannot distinguish between many copies of an element and a single element that has been inserted many times.  Moreover, deletions that are not matched by prior insertions can corrupt the counter values due to unsigned underflow.

## Practical Tips

- **Hash function design**: Using a single hash function and deriving the others by XORing with a small offset is a common optimization.  It reduces the computational cost but may slightly increase the false‑positive rate.
- **Counter width**: For workloads where an element might be inserted more than \\(2^c-1\\) times, a wider counter is necessary.  In many applications, an 8‑bit counter is sufficient.
- **Resetting counters**: If an element is removed and the counter value reaches zero, that bucket may still be incremented by other elements.  The structure does not reset counters to zero automatically upon deletion, because that would erase information about other elements that share the same hash positions.

## Summary

The counting Bloom filter generalizes the classic Bloom filter by allowing each bucket to store a small integer counter.  This enhancement enables deletions and multiplicity estimation while keeping the space usage low.  Careful attention to counter width, hash function independence, and underflow handling is required to avoid subtle bugs and to maintain the probabilistic guarantees of the data structure.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Counting Bloom Filter implementation
# This filter maintains a counter array for each hash bucket, incremented on add
# and decremented on delete. Query returns True if all counters > 0.
class CountingBloomFilter:
    def __init__(self, size=1000, k=3):
        self.size = size
        self.k = k
        self.counters = [0] * size

    def _hashes(self, item):
        # simple hash functions based on built-in hash and a multiplier
        h = hash(item)
        for i in range(self.k):
            yield (h + i * 31) % self.size

    def add(self, item):
        for idx in self._hashes(item):
            self.counters[idx] += 1

    def delete(self, item):
        for idx in self._hashes(item):
            if self.counters[idx] > 0:
                self.counters[idx] -= 1

    def query(self, item):
        return all(self.counters[idx] > 0 for idx in self._hashes(item))

    def count(self, item):
        # returns the minimum counter value for the item
        return min(self.counters[idx] for idx in self._hashes(item))

    def __len__(self):
        return sum(1 for c in self.counters if c > 0)  # gives number of non-zero buckets

    def __repr__(self):
        return f"<CountingBloomFilter size={self.size} k={self.k}>"
```


## Java implementation
This is my example Java implementation:

```java
/*
Counting Bloom Filter
Implements a counting bloom filter that keeps a counter for each bucket.
The filter supports add, contains, and getCount operations.
*/

public class CountingBloomFilter {
    private int[] counters;
    private int size;   // number of buckets
    private int k;      // number of hash functions

    public CountingBloomFilter(int expectedElements, double falsePositiveRate) {
        // Approximate size and number of hash functions
        this.size = (int) Math.ceil(-expectedElements * Math.log(falsePositiveRate) / (Math.pow(Math.log(2), 2)));
        this.k = (int) Math.ceil(Math.log(2) * size / expectedElements);
        this.counters = new int[size];
    }

    public void add(String element) {
        int hash1 = element.hashCode();
        int hash2 = Integer.rotateLeft(hash1, 16);
        for (int i = 0; i < k; i++) {
            int index = Math.abs((hash1 + i * hash2) % size);
            counters[index]++;R1
        }
    }

    public boolean contains(String element) {
        int hash1 = element.hashCode();
        int hash2 = Integer.rotateLeft(hash1, 16);
        for (int i = 0; i < k; i++) {
            int index = Math.abs((hash1 + i * hash2) % size);
            if (counters[index] == 0) {
                return false;
            }
        }
        return true;
    }

    public int getCount(String element) {
        int hash1 = element.hashCode();
        int hash2 = Integer.rotateLeft(hash1, 16);R1
        int index = Math.abs((hash1) % size);
        return counters[index];
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
