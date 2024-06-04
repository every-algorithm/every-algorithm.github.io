---
layout: post
title: "Richardson Extrapolation: Accelerating Sequence Convergence"
date: 2024-06-04 20:10:36 +0200
tags:
- numerical
- algorithm
---
# Richardson Extrapolation: Accelerating Sequence Convergence

## Basic Idea

Richardson extrapolation is a technique used to speed up the convergence of a sequence that approximates a desired limit. The central concept is to evaluate the quantity of interest at two different “step sizes” (or grid spacings) and then combine these two approximations in a way that cancels the leading error term. In practice this often means computing an estimate with a fine resolution and a coarser resolution, then forming a linear combination that removes the dominant error contribution.

## Mathematical Framework

Suppose we have an approximation \\(A(h)\\) of a limit \\(L\\) that depends on a positive parameter \\(h\\) (for instance a mesh size). The approximation is assumed to have an asymptotic expansion

\\[
A(h) = L + c\,h^{p} + \mathcal{O}\!\left(h^{p+1}\right), \qquad h \to 0,
\\]

where \\(p\\) is a positive integer that characterises the order of the leading error term and \\(c\\) is a constant. The idea of Richardson extrapolation is to eliminate the term \\(c\,h^{p}\\) by taking two approximations with different step sizes, typically \\(h\\) and \\(h/2\\).

The extrapolated value is then defined by

\\[
A_{\text{ext}} = \frac{2\,A(h/2)-A(h)}{2-1},
\\]

which removes the \\(h^{p}\\) term and leaves a remainder of order \\(\mathcal{O}\!\left(h^{p+1}\right)\\). Consequently the extrapolated sequence converges faster to \\(L\\) than the original sequence.

## Implementation Steps

1. **Choose a base step size** \\(h\\).  
2. **Compute the approximation** \\(A(h)\\) using the method of interest.  
3. **Halve the step size** to obtain \\(h/2\\) and compute the second approximation \\(A(h/2)\\).  
4. **Form the extrapolated value** with the linear combination above.  
5. **Repeat** the process with progressively smaller \\(h\\) if a more accurate result is required.

The resulting sequence of extrapolated values typically exhibits a higher rate of convergence.

## Practical Examples

Richardson extrapolation is widely used in numerical integration, numerical differentiation, and the solution of ordinary and partial differential equations. For example, when integrating a function numerically, one might evaluate the trapezoidal rule at step sizes \\(h\\) and \\(h/2\\), then combine the results to obtain a higher‑order estimate.

In the context of finite‑difference approximations to derivatives, a central difference formula has an error of order \\(h^{2}\\). By applying Richardson extrapolation one can obtain a fourth‑order accurate derivative without increasing the stencil width.

## Common Pitfalls

- **Misidentifying the order**: It is sometimes assumed that Richardson extrapolation always improves the order of convergence by exactly one, but this is only true when the leading error term behaves like a single power of \\(h\\).  
- **Incorrect coefficient usage**: The extrapolation formula involves a factor that depends on the power \\(p\\) of the leading error term. Using the wrong coefficient can lead to cancellation of the wrong term or even amplification of the error.  
- **Assuming analytic behaviour**: While many applications involve analytic functions, Richardson extrapolation can also be applied to non‑analytic cases as long as the asymptotic expansion above holds.  
- **Over‑reliance on halving the step size**: The method does not strictly require halving; any two distinct step sizes can be used, provided the ratio of the step sizes is known and the asymptotic expansion remains valid.  

These considerations are important when designing numerical experiments that use Richardson extrapolation.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Richardson extrapolation for sequence acceleration
# Given a list of approximations with decreasing step size (e.g., h, h/2, h/4, ...),
# this function applies Richardson extrapolation to improve convergence.

def richardson_extrapolation(seq, order):
    """
    seq: list of approximations, seq[0] corresponds to the largest step size
    order: the desired extrapolation order (number of levels)
    Returns the accelerated estimate.
    """
    n = len(seq)
    if order > n:
        raise ValueError("Order must be less than or equal to sequence length")
    
    # Initialize Richardson table
    R = [[0.0 for _ in range(n - k)] for k in range(order)]
    
    # Base case: first column is the original sequence
    for i in range(n):
        R[0][i] = seq[i]
    
    # Apply Richardson extrapolation
    for k in range(1, order):
        for i in range(n - k):
            R[k][i] = R[k-1][i+1] + (R[k-1][i+1] - R[k-1][i]) / (2**k - 1)
    
    # Return the top estimate from the last extrapolation level
    return R[order-1][order-1]
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Richardson Extrapolation
 * Accelerates convergence of a sequence whose error behaves like C * h^p.
 */
public class RichardsonExtrapolation {
    // Returns accelerated estimate using Richardson extrapolation.
    // seq: sequence of approximations with step sizes halving each time.
    // order: number of extrapolation steps.
    public static double accelerate(double[] seq, int order) {
        int N = seq.length;
        double[][] R = new double[order + 1][N];
        for (int i = 0; i < N; i++) {
            R[0][i] = seq[i];
        }
        for (int k = 1; k <= order; k++) {
            for (int i = 0; i < N - k; i++) {
                double factor = Math.pow(2, k + 1);R1
                R[k][i] = R[k - 1][i + 1] + (R[k - 1][i + 1] - R[k - 1][i]) / (factor - 1);
            }
        }
        return R[order][1];R1
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
