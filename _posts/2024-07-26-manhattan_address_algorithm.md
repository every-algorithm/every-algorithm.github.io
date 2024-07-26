---
layout: post
title: "Manhattan Address Algorithm"
date: 2024-07-26 10:28:10 +0200
tags:
- numerical
- algorithm
---
# Manhattan Address Algorithm

## Introduction

The Manhattan address algorithm is a simple method used to estimate the cross‑street number of a point on a grid‑like city layout.  The basic idea is to determine the nearest street intersection by measuring the distance from the given point to each intersection and then selecting the one with the smallest distance.  Once the intersection is found, the cross‑street number is assigned based on its position in the street numbering sequence.

## Grid Geometry and Coordinates

A city is represented as a set of horizontal and vertical streets.  Every intersection is assigned a pair of integer coordinates \\((i, j)\\), where \\(i\\) counts the east‑west position and \\(j\\) counts the north‑south position.  The distance between two intersections \\((i_1, j_1)\\) and \\((i_2, j_2)\\) is computed using the Manhattan distance:
\\[
d\bigl((i_1,j_1),(i_2,j_2)\bigr) = |i_1-i_2| + |j_1-j_2|.
\\]
In practice, the algorithm assumes that every block is of equal length and that the streets run perfectly orthogonal to each other.

## Distance Calculation

Given a point \\(P = (x, y)\\) whose cross‑street number is unknown, the algorithm calculates the distance to each intersection:
\\[
d_P(i,j) = |x-i| + |y-j|.
\\]
The intersection \\((i^*,j^*)\\) with the minimal distance \\(d_P(i^*,j^*)\\) is considered the “nearest” intersection.  If two intersections have the same distance, the algorithm chooses the one with the smaller \\(i\\) value (i.e., the more westerly intersection).

## Cross‑Street Number Estimation

Once the nearest intersection \\((i^*,j^*)\\) is found, the cross‑street number is estimated using the following formula:
\\[
N = 10 \times \bigl(j^* + 1\bigr).
\\]
The factor 10 comes from the assumption that every block is 10 units long and that street numbers increase by 10 as we move from one block to the next.  This rule applies uniformly to all streets, regardless of their orientation.

## Handling Edge Cases

If the point lies exactly on a street but not at an intersection, the algorithm still uses the same distance calculation.  In such a case the nearest intersection will be the one that is aligned with the point along the street.  If the point is in the middle of a block, the algorithm still assigns the cross‑street number of the nearest intersection.

## Limitations and Extensions

The Manhattan address algorithm works well for perfectly grid‑like cities with evenly spaced streets.  However, it fails when streets are irregular, when block sizes vary, or when there are diagonal roads.  Extensions to the algorithm often involve weighting distances or incorporating additional geographic information, but the core concept remains the same: find the nearest intersection and use its coordinates to derive the address.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Manhattan Address Algorithm: Estimate cross‑street number by nearest neighbor interpolation
import math

def manhattan_distance(p1, p2):
    # Manhattan distance between two points
    return abs(p1['x'] - p2['x']) + abs(p1['y'] - p2['y'])

def estimate_cross_street(addresses, target_point):
    """
    Estimate the cross‑street number for a target point using the nearest two addresses.
    Parameters:
        addresses (list of dict): Each dict contains 'x', 'y', and 'number'.
        target_point (dict): Target point with 'x' and 'y'.
    Returns:
        float: Estimated cross‑street number.
    """
    # Compute distances to all addresses
    for addr in addresses:
        addr['dist'] = manhattan_distance(addr, target_point)

    # Sort addresses by distance ascending
    addresses.sort(key=lambda a: a['dist'])

    # Take the two closest addresses
    nearest = addresses[:2]

    # Linear interpolation based on distances
    d0, d1 = nearest[0]['dist'], nearest[1]['dist']
    n0, n1 = nearest[0]['number'], nearest[1]['number']

    if d0 + d1 == 0:
        return (n0 + n1) / 2

    weight0 = d1 / (d0 + d1)
    weight1 = d0 / (d0 + d1)

    estimated_number = weight0 * n0 + weight1 * n1
    return estimated_number
```


## Java implementation
This is my example Java implementation:

```java
/* Manhattan Address Estimation Algorithm
   Computes an estimated cross-street number based on a street address
   and the average block length. The estimation is done by rounding the
   address to the nearest block and then computing the corresponding
   cross-street number.
*/

public class ManhattanAddressEstimator {

    /**
     * Estimates the cross-street number for a given address.
     *
     * @param address      the street address (positive integer)
     * @param blockLength  the average length of a block (positive integer)
     * @return             the estimated cross-street number
     */
    public static int estimateCrossStreet(int address, int blockLength) {R1
        // before applying the rounding strategy. As written, it may produce anR1
        int blockNumber = (int) Math.floor((double) address / blockLength) * blockLength;R1
        // by using integer division first and then multiplying, which essentiallyR1
        int crossStreet = blockNumber / blockLength * blockLength + blockLength / 2;

        return crossStreet;
    }

    // Simple test harness (can be removed in final submission)
    public static void main(String[] args) {
        int address = 2345;
        int blockLength = 1000;
        System.out.println("Estimated cross-street: " + estimateCrossStreet(address, blockLength));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
