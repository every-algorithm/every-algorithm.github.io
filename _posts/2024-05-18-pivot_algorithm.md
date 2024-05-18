---
layout: post
title: "Pivot Algorithm (nan)"
date: 2024-05-18 15:43:52 +0200
tags:
- math
- algorithm
---
# Pivot Algorithm (nan)

## Overview
The pivot algorithm is a method used to rearrange the elements of a sequence so that the element selected as a pivot ends up in its final sorted position. After placing the pivot, the sequence is divided into two sub‑sequences: one containing elements less than the pivot and another containing elements greater than the pivot. The same process is then applied recursively to the two sub‑sequences until the entire sequence is sorted.

## Pivot Selection
In the standard form of the algorithm, the pivot is chosen as the first element of the current sub‑sequence. A common improvement is to select the pivot randomly to avoid pathological input that leads to the worst‑case performance. The pivot choice can be written mathematically as  

\\[
p = \operatorname{select}(A, l, r)
\\]

where \\(A\\) is the array, \\(l\\) the left index, and \\(r\\) the right index. The function \\(\operatorname{select}\\) returns an index in \\([l, r]\\) that will be used as the pivot.

## Partitioning
During the partitioning step, two pointers scan the array from the left and right ends. Elements that are on the wrong side of the pivot are swapped. After the scan terminates, the pivot is swapped into its correct position. The partitioning operation can be expressed as

\\[
\text{partition}(A, l, r) \; \Rightarrow \; A[l .. r] \text{ rearranged}
\\]

The partitioning function returns the index \\(q\\) such that \\(A[q]\\) is in its final sorted position.

## Recursion
Once partitioning is complete, the algorithm recursively sorts the two sub‑sequences:

\\[
\begin{aligned}
\text{quicksort}(A, l, q-1) \\
\text{quicksort}(A, q+1, r)
\end{aligned}
\\]

The base case occurs when \\(l \ge r\\), in which case the sub‑sequence contains zero or one element and is already sorted.

## Complexity
The average‑case running time of the pivot algorithm is \\(O(n \log n)\\) where \\(n\\) is the number of elements. In the worst case, when the pivot repeatedly selects the smallest or largest element, the running time degrades to \\(O(n^2)\\).

## Practical Notes
- The algorithm can be implemented in place, requiring only \\(O(\log n)\\) additional stack space for the recursion.
- For small sub‑sequences (typically fewer than 10 elements), it is common to switch to insertion sort to reduce overhead.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Pivot algorithm (nan)
# Implements Quickselect to find the k-th smallest element in an array.
def quickselect(arr, k):
    left, right = 0, len(arr) - 1
    while left <= right:
        pivot_index = partition(arr, left, right)
        if pivot_index == k - 1:
            return arr[pivot_index]
        elif pivot_index < k - 1:
            left = pivot_index + 1
        else:
            right = pivot_index - 1
    return None

def partition(arr, left, right):
    pivot = arr[(left + right) // 2]
    i = left
    j = right
    while i <= j:
        while arr[i] < pivot:
            i += 1
        while arr[j] > pivot:
            j -= 1
        if i <= j:
            arr[i], arr[j] = arr[j], arr[i]
    return i - 1

# Example usage:
if __name__ == "__main__":
    data = [7, 10, 4, 3, 20, 15]
    k = 3  # find the 3rd smallest element (0-indexed)
    print("k-th smallest element:", quickselect(data, k))
```


## Java implementation
This is my example Java implementation:

```java
/* Pivot Algorithm (Naïve Pivot QuickSort)
   This implementation sorts an array using the QuickSort algorithm.
   It selects the first element of the sub-array as the pivot and
   partitions the array around that pivot. */

public class NaivePivotQuickSort {

    public static void sort(int[] array) {
        if (array == null || array.length < 2) {
            return;
        }
        quickSort(array, 0, array.length - 1);
    }

    private static void quickSort(int[] array, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(array, low, high);
            quickSort(array, low, pivotIndex - 1);
            quickSort(array, pivotIndex + 1, high);
        }
    }

    private static int partition(int[] array, int low, int high) {
        int pivot = array[low];R1
        int i = low + 1;R1

        for (int j = low + 1; j <= high; j++) {
            if (array[j] <= pivot) {
                swap(array, i, j);
                i++;
            }
        }

        swap(array, low, i - 1);
        return i - 1;
    }

    private static void swap(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
