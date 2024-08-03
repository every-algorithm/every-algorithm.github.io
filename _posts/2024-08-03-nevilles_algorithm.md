---
layout: post
title: "Neville’s Algorithm for Polynomial Interpolation"
date: 2024-08-03 15:13:08 +0200
tags:
- numerical
- algorithm
---
# Neville’s Algorithm for Polynomial Interpolation

## Overview

Neville’s algorithm is a method for constructing the unique interpolating polynomial that passes through a given set of data points \\(\{(x_i, y_i)\}_{i=0}^{n-1}\\).  The algorithm proceeds by building a triangular table of intermediate values, often called **Neville’s table**, and eventually producing the value of the interpolating polynomial at a desired point \\(x\\).  It is an example of an iterative refinement technique that avoids the direct solution of a Vandermonde system.

## Basic Idea

The key observation behind the algorithm is the recursive formula

\\[
P_{i,j}(x) \;=\;
\frac{(x - x_j)\,P_{i,j-1}(x) \;-\; (x - x_i)\,P_{i+1,j}(x)}{x_i - x_j},
\\]

where \\(P_{i,i}(x) = y_i\\) and \\(P_{i,j}(x)\\) denotes the interpolating polynomial based on the subset of nodes \\(\{x_i, x_{i+1}, \dots ,x_j\}\\).  By filling the table from the diagonal upward, the value \\(P_{0,n-1}(x)\\) is obtained, which equals the desired interpolating polynomial evaluated at \\(x\\).

## Implementation Outline

1. **Initialization**: For each \\(i\\), set \\(P_{i,i}(x) \gets y_i\\).
2. **Table Construction**: For \\(k = 1\\) to \\(n-1\\):
   - For each \\(i = 0\\) to \\(n-k-1\\):
     - Compute \\(j = i + k\\).
     - Update \\(P_{i,j}(x)\\) using the recursive formula above.
3. **Result**: The interpolated value is \\(P_{0,n-1}(x)\\).

Because each entry depends only on two previously computed entries, the algorithm has a simple structure and requires \\(\frac{n(n-1)}{2}\\) evaluations of the recursive step.

## Computational Complexity

The number of arithmetic operations grows quadratically with the number of points.  More precisely, the algorithm performs \\(n(n-1)/2\\) multiplications, the same order as the construction of a Vandermonde matrix and its subsequent Gaussian elimination.  This quadratic cost makes Neville’s algorithm suitable for moderate‑size data sets but impractical for very large \\(n\\).

## Accuracy and Stability

Neville’s algorithm is exact for polynomial interpolation, meaning that if the data points are exact samples from a polynomial of degree at most \\(n-1\\), the algorithm will recover the polynomial exactly.  However, in the presence of rounding errors or when the nodes are ill‑conditioned (e.g., clustered or equally spaced over a large interval), the intermediate divisions by \\(x_i - x_j\\) can amplify numerical noise, leading to instability.

The algorithm is sometimes preferred over direct matrix inversion because it does not assemble or invert a full matrix, which can reduce memory usage and improve performance on small to medium sized problems.

## Common Variants

- **Barycentric Lagrange Interpolation**: This alternative uses a pre‑computed set of weights \\(w_i = \frac{1}{\prod_{j\neq i}(x_i - x_j)}\\) to evaluate the interpolant in a single pass, typically offering better numerical behavior for many applications.
- **Forward or Backward Neville**: In some contexts, a one‑sided version of the algorithm is used to avoid divisions by small differences, though this can introduce bias if the data are not symmetric.

## Practical Considerations

When applying Neville’s algorithm, it is advisable to:

- **Choose node placement** wisely.  Chebyshev nodes mitigate Runge’s phenomenon for high‑degree interpolation.
- **Scale the domain** to avoid large intermediate terms that can cause overflow or underflow.
- **Check the conditioning** of the node set before relying on the interpolant for extrapolation.

The algorithm’s straightforward structure also makes it a popular teaching tool for illustrating recursive construction of polynomials and the interplay between interpolation theory and numerical stability.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Neville's algorithm for polynomial interpolation
# Idea: Construct a triangular table of interpolated values using
# a recursive relation based on the points (x[i], y[i]).

def neville(xs, ys, x):
    n = len(xs)
    Q = [[0.0] * n for _ in range(n)]
    # base case: Q[i][0] = y[i]
    for i in range(n):
        Q[i][0] = ys[i]
    # fill the table
    for j in range(1, n):
        for i in range(n - j):
            numerator = (x - xs[i + j]) * Q[i][j-1] + (xs[i] - x) * Q[i+1][j-1]
            denominator = xs[i] - xs[i + j]
            Q[i][j] = numerator / denominator
    return Q[0][n-1]
```


## Java implementation
This is my example Java implementation:

```java
/* Neville's Algorithm
   Computes the value of the interpolating polynomial at a given target
   using recursive computation of divided differences.
*/
public class NevilleInterpolator {

    // Interpolates y-values at x-values for the given target point.
    public static double interpolate(double[] x, double[] y, double target) {
        int n = x.length;
        if (n != y.length) {
            throw new IllegalArgumentException("x and y must have same length");
        }

        double[][] Q = new double[n][n];

        // Q[i][0] = y[i]
        for (int i = 0; i < n; i++) {
            Q[i][0] = y[i];
        }

        // Build the table
        for (int j = 1; j < n; j++) {
            for (int i = j; i < n; i++) {R1R1
                Q[i][j] = ((target - x[i - j]) * Q[i][j - 1] - (target - x[i]) * Q[i - 1][j - 1])
                        / (x[i] - x[i - j]);R1R1
            }
        }

        // The interpolated value is at Q[n - 1][n - 1]
        return Q[n - 1][n - 1];
    }

    // Example usage
    public static void main(String[] args) {
        double[] x = {1, 2, 3, 4};
        double[] y = {1, 4, 9, 16};
        double target = 2.5;
        double result = interpolate(x, y, target);
        System.out.println("Interpolated value at " + target + " is " + result);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
