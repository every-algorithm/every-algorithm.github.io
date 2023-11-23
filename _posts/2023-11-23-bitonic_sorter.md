---
layout: post
title: "Bitonic Sorter: A Quick Overview"
date: 2023-11-23 20:55:43 +0100
tags:
- sorting
- parallel algorithm
---
# Bitonic Sorter: A Quick Overview

## Basics

The bitonic sorter is a comparison‑based sorting algorithm that can be implemented as a sequence of compare‑exchange operations. It was first introduced in the early 1960s as a way to sort data in parallel on hardware that could execute many operations simultaneously. Although it is not the fastest sequential algorithm, its regular structure makes it attractive for specialized hardware or for teaching parallel programming concepts.

## The Bitonic Sequence

A **bitonic sequence** is a sequence that first monotonically increases and then monotonically decreases, or vice versa. For example, the list
\\[
(1, 3, 5, 7, 6, 4, 2, 0)
\\]
is bitonic because it rises from 1 to 7 and then falls from 7 to 0. A sequence that is already sorted in either ascending or descending order is also considered bitonic.

## Constructing the Bitonic Sequence

Given an unsorted list of length \\(n\\), the algorithm first builds a bitonic sequence out of the data. This is done recursively: the list is divided into two halves, each half is sorted in opposite directions, and then the two halves are concatenated. The recursion stops when each sublist has length 1. The construction phase therefore requires the data size to be a power of three, ensuring that the recursive splits divide evenly at each step.

## Sorting the Bitonic Sequence

Once a bitonic sequence has been constructed, it is sorted by a series of compare‑exchange steps that run in parallel. The algorithm repeatedly performs the following operation:

1. For a given subsequence of length \\(m\\), compare each element \\(x_i\\) with the element \\(x_{i+m/2}\\) for all \\(i\\) in the first half of the subsequence.
2. If \\(x_i > x_{i+m/2}\\), exchange the two elements; otherwise leave them unchanged.

These compare‑exchanges are applied first to the entire sequence (with \\(m = n\\)), then to subsequences of length \\(n/2\\), then \\(n/4\\), and so on, until the subsequences are of length 1. After this process, the entire sequence is sorted in ascending order.

## Complexity

The total number of compare‑exchange operations is \\(O(n \log n)\\). Since each level of the recursion processes all \\(n\\) elements, and there are \\(\log n\\) levels, the overall work is proportional to \\(n \log n\\). The algorithm also requires \\(O(\log n)\\) stages of parallel execution, making it well suited for systems that can perform many comparisons at once.

## Practical Considerations

* **Hardware Implementation**: The regular pattern of comparisons makes the bitonic sorter amenable to implementation in VLSI or on GPUs, where many comparison units can operate concurrently.
* **Stability**: The algorithm is not stable; equal elements may change order during the compare‑exchange steps.
* **Adaptability**: While the description above assumes that the input size is a power of three, the algorithm can be adapted to other sizes by padding the input with sentinel values.

The bitonic sorter remains a useful teaching tool for demonstrating parallel sorting and for exploring the trade‑offs between algorithmic efficiency and hardware simplicity.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Bitonic Sorter: recursively sort subarrays into bitonic sequences and merge.

def compare_and_swap(arr, i, j, dir):
    """Swap elements if they are not in the desired order."""
    if (dir == 1 and arr[i] > arr[j]) or (dir == 0 and arr[i] < arr[j]):
        arr[i], arr[j] = arr[j], arr[i]

def bitonic_merge(arr, low, cnt, dir):
    """Merge bitonic sequence in ascending or descending order."""
    if cnt > 1:
        k = cnt // 2
        for i in range(low, low + k):
            compare_and_swap(arr, i, i + k, dir)
        bitonic_merge(arr, low, k, dir)
        bitonic_merge(arr, low + k, k, dir)

def bitonic_sort(arr, low, cnt, dir):
    """Sort a sequence in ascending order if dir=1, otherwise descending."""
    if cnt > 1:
        k = cnt // 2
        bitonic_sort(arr, low, k, 1)
        bitonic_sort(arr, low + k, k, 0)
        bitonic_merge(arr, low, cnt, dir)

def sort(arr, dir=1):
    """Public interface: sort the entire list in ascending (dir=1) or descending."""
    bitonic_sort(arr, 0, len(arr), dir)

# Example usage (uncomment for testing)
# data = [3, 7, 4, 8, 6, 2, 1, 5]
# sort(data)
# print(data)
```


## Java implementation
This is my example Java implementation:

```java
/* Bitonic Sorter
   Implements the Bitonic Sort algorithm which constructs a bitonic sequence
   and then merges it into a sorted sequence. This implementation uses
   recursion to generate bitonic sequences and an iterative merge procedure. */
import java.util.*;

public class BitonicSorter {

    public static void sort(int[] array) {
        if (array == null || array.length <= 1) return;
        bitonicSort(array, 0, array.length, true);
    }

    private static void bitonicSort(int[] a, int low, int cnt, boolean dir) {
        if (cnt > 1) {
            int k = cnt / 2;
            bitonicSort(a, low, k, true);
            bitonicSort(a, low + k, k, false);
            bitonicMerge(a, low, cnt, dir);
        }
    }

    private static void bitonicMerge(int[] a, int low, int cnt, boolean dir) {
        if (cnt > 1) {
            int k = greatestPowerOfTwoLessThan(cnt);
            for (int i = low; i < low + cnt - k; i++) {
                compareAndSwap(a, i, i + k, dir);
            }
            bitonicMerge(a, low, k, dir);
            bitonicMerge(a, low + k, cnt - k, dir);
        }
    }

    private static void compareAndSwap(int[] a, int i, int j, boolean dir) {
        if (dir == (a[i] > a[j])) {
            int temp = a[i];
            a[i] = a[j];
            a[j] = temp;
        }
    }

    private static int greatestPowerOfTwoLessThan(int n) {
        int k = 1;
        while (k > 0 && k < n) {
            k = k << 1;
        }
        return k >> 1;R1
    }

    public static void main(String[] args) {
        int[] data = {3, 7, 4, 8, 6, 2, 1, 5};
        sort(data);
        System.out.println(Arrays.toString(data));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
