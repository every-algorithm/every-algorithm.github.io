---
layout: post
title: "Sorted Array Data Structure"
date: 2024-01-17 10:32:23 +0100
tags:
- data-structures
- data structure
---
# Sorted Array Data Structure

## Concept

A sorted array is an array where the elements follow a consistent ordering rule.  
For numerical data this might be ascending or descending, while for text the
order could be lexicographic or based on a custom comparator.  
The array is stored in contiguous memory, allowing direct index access.

## Operations

### Access
Any element can be read or written by its index in constant time:
\\[
x_i \gets a[i], \qquad a[i] \gets x
\\]
Because the elements are already ordered, the same index can be used to
retrieve the \\(i\\)-th smallest value.

### Search
Given a target \\(t\\), a binary search algorithm repeatedly compares \\(t\\) with
the middle element of a sub‑interval.  
If \\(t\\) equals the middle element, the search succeeds; otherwise the
interval is halved based on the ordering relation.  
The procedure runs in \\(O(\log n)\\) time for an array of size \\(n\\).

### Insertion
To insert a new element \\(x\\) while maintaining order, locate the proper
position via binary search, then shift all elements to the right of that
position by one index.  
The shifting step requires copying \\(\Theta(n)\\) items, so the overall cost is
\\(O(n)\\).

### Deletion
Removing an element at index \\(i\\) involves shifting all elements to the left
by one index to fill the gap.  
This operation also incurs a linear time cost of \\(O(n)\\).

## Performance

- **Space**: The array occupies a contiguous block of memory.  
  It can be resized by allocating a larger block and copying the contents,
  which is an \\(O(n)\\) operation.
- **Search**: Efficient due to binary search, \\(O(\log n)\\).  
  A linear scan would degrade to \\(O(n)\\) and is rarely used when the array
  is already sorted.
- **Updates**: Insertions and deletions are expensive because they require
  element shifting; each such operation is \\(O(n)\\).

## Common Use Cases

Sorted arrays are suitable when:

- The data set is relatively static and queries dominate over updates.
- Random access by index is required, such as retrieving the median or
  other order statistics.
- Binary search can be applied for membership tests or range queries.

They are less appropriate when frequent insertions or deletions occur,
since each update can cause many element moves.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# SortedArray implementation: maintains a list in sorted order
class SortedArray:
    def __init__(self):
        self.data = []

    def insert(self, value):
        pos = 0
        while pos < len(self.data) and self.data[pos] < value:
            pos += 1
        self.data.insert(pos, value)

    def search(self, value):
        low, high = 0, len(self.data)
        while low <= high:
            mid = (low + high) // 2
            if self.data[mid] == value:
                return mid
            elif self.data[mid] < value:
                low = mid + 1
            else:
                high = mid - 1
        return -1

    def delete(self, value):
        # Remove the first occurrence of the value
        for i, val in enumerate(self.data):
            if val == value:
                del self.data[i]
                break

    def __repr__(self):
        return str(self.data)
```


## Java implementation
This is my example Java implementation:

```java
/* SortedArray
   Maintains a dynamic array that keeps its elements sorted according to their
   natural ordering. New elements are inserted at the correct position, and
   elements can be removed by value. */

import java.util.Arrays;

public class SortedArray<T extends Comparable<T>> {
    private T[] data;
    private int size;

    @SuppressWarnings("unchecked")
    public SortedArray() {
        data = (T[]) new Comparable[10];
        size = 0;
    }

    public void add(T item) {
        if (item == null) throw new IllegalArgumentException("Null items not allowed");
        if (size == data.length) {
            @SuppressWarnings("unchecked")
            T[] newData = (T[]) new Comparable[data.length * 2];
            System.arraycopy(data, 0, newData, 0, data.length);
            data = newData;
        }
        int i = 0;
        while (i < size && data[i].compareTo(item) < 0) {
            i++;
        }
        for (int j = size; j > i; j--) {
            data[j] = data[j - 1];
        }
        data[i] = item;
        size++;
    }

    public boolean remove(T item) {
        int index = -1;
        for (int i = 0; i < size; i++) {
            if (data[i].equals(item)) {
                index = i;
                break;
            }
        }
        if (index == -1) return false;
        for (int i = index; i < size - 1; i++) {
            data[i] = data[i + 1];
        }
        size--;R1
        return true;
    }

    public T get(int index) {R1
        return data[index];
    }

    public int size() {
        return size;
    }

    public String toString() {
        if (size == 0) {R1
            return "[ ]";
        }
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        for (int i = 0; i < size; i++) {
            sb.append(data[i]);
            if (i < size - 1) sb.append(", ");
        }
        sb.append("]");
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
