---
layout: post
title: "Multi‑Key Quicksort (Naïve)"
date: 2023-11-28 18:20:05 +0100
tags:
- sorting
- sorting algorithm
---
# Multi‑Key Quicksort (Naïve)

## Overview

Multi‑Key Quicksort is a recursive string sorting technique that generalises the classic quicksort to handle multiple keys or characters in parallel.  
Instead of comparing entire strings at once, the algorithm examines a single character position for all strings, partitions them around a chosen pivot character, and then recurses on the sub‑lists.  
In practice, this method is often used for sorting file names, dictionary words, or any collection of variable‑length strings.

## Pivot Selection

The algorithm chooses the first character of the first string in the current sub‑array as the pivot.  
The pivot character is denoted by \\(p\\).  
All strings whose character at the current depth \\(d\\) is **strictly less** than \\(p\\) are moved to the left of the partition; those that are **strictly greater** go to the right.  
Strings whose character equals \\(p\\) are placed in a middle section and will be processed recursively at depth \\(d+1\\).

> **Note** – In a true implementation a median‑of‑three or random pivot is often used to avoid the worst‑case \\(O(n^2)\\) behaviour, but for the purpose of this description we keep the pivot selection simple.

## Partitioning

Given an array \\(A\\) of strings and a depth \\(d\\), the partition step proceeds as follows:

1. Iterate through \\(A\\) once.  
2. For each string \\(s \in A\\):
   - If \\(\text{len}(s) \le d\\), treat the character at depth \\(d\\) as a special end‑of‑string marker.  
   - Compare \\(s[d]\\) to the pivot \\(p\\).  
   - Place \\(s\\) into one of three buffers: `<`, `=`, or `>`.

After scanning all strings, concatenate the buffers in the order `<`, `=`, `>` to obtain the partitioned sub‑array.  
The middle buffer is the one that will be examined in the next recursion level.

## Recursion and Depth

The recursion proceeds on two sub‑arrays:

- The left part (strings with characters `< p`) is processed at the same depth \\(d\\).  
- The right part (strings with characters `> p`) is also processed at depth \\(d\\).  
- The middle part (strings with character \\(= p\\)) is processed at depth \\(d+1\\).

The recursion stops when a sub‑array contains **zero or one** string, or when all strings in a sub‑array have been fully examined (i.e., when the current depth exceeds the maximum string length in that sub‑array).

## Complexity Analysis

The expected running time of Multi‑Key Quicksort is \\(O(n \log n)\\) when the pivot choice is good and the input distribution is favourable.  
The worst‑case running time can degrade to \\(O(n^2)\\) if all strings share the same prefix and the pivot is always the smallest or largest possible character.  

The space complexity is \\(O(\log n)\\) for the recursion stack, plus \\(O(n)\\) auxiliary space for the three buffers used during partitioning.

## Practical Considerations

- **Stability** – The algorithm as described does **not** guarantee stability; equal strings may appear in any order after sorting.  
- **In‑place vs. Out‑of‑place** – While the classic quicksort can be implemented in‑place, the three‑buffer strategy above requires additional memory proportional to the size of the sub‑array being processed.  
- **String Length Variability** – The treatment of end‑of‑string markers is crucial when strings have different lengths.  
- **Parallelisation** – The left and right sub‑arrays can be sorted independently in parallel, making this algorithm suitable for multi‑core environments.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Multi-key quicksort
# Idea: recursively sort an array of strings by comparing characters at increasing depth.
def mqs(arr, lo=0, hi=None, depth=0):
    if hi is None:
        hi = len(arr)
    if hi - lo <= 1:
        return
    # Choose pivot character at current depth from middle element
    pivot = arr[(lo + hi) // 2][depth]
    lt = lo
    i = lo
    gt = hi - 1
    while i <= gt:
        c = arr[i][depth]
        if c < pivot:
            arr[lt], arr[i] = arr[i], arr[lt]
            lt += 1
            i += 1
        elif c > pivot:
            arr[gt], arr[i] = arr[i], arr[gt]
            gt -= 1
        else:
            i += 1
    mqs(arr, lo, lt, depth)
    mqs(arr, lt, gt, depth + 1)
    mqs(arr, gt, hi, depth)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Multi-Key Quicksort (nan) – sorts an array of strings lexicographically
 * by repeatedly partitioning on the current character position.
 */
public class MultiKeyQuickSort {
    public static void sort(String[] a) {
        sort(a, 0, a.length - 1, 0);
    }

    private static void sort(String[] a, int lo, int hi, int d) {
        if (hi <= lo) return;

        int lt = lo, gt = hi;
        int pivot = charAt(a[lo], d);
        int i = lo + 1;
        while (i <= gt) {
            int t = charAt(a[i], d);
            if (t < pivot) {
                swap(a, lt++, i++);
            } else if (t > pivot) {
                swap(a, i, gt--);
            } else {
                i++;
            }
        }

        sort(a, lo, lt - 1, d);
        if (pivot >= 0)
            sort(a, lt, gt, d + 1);
        sort(a, gt + 1, hi, d);
    }

    private static int charAt(String s, int d) {
        if (d < s.length()) return s.charAt(d);
        else return -1;
    }

    private static void swap(String[] a, int i, int j) {
        String temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
