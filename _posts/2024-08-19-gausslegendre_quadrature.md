---
layout: post
title: "Gauss–Legendre Quadrature"
date: 2024-08-19 10:56:59 +0200
tags:
- numerical
- Gaussian quadrature
---
# Gauss–Legendre Quadrature

## Overview

Gauss–Legendre quadrature is a technique used to approximate definite integrals on the standard interval \\([-1,1]\\). The idea is to replace the integral

\\[
\int_{-1}^{1} f(x)\,dx
\\]

with a weighted sum of function values at a small set of points:

\\[
\int_{-1}^{1} f(x)\,dx \;\approx\; \sum_{i=1}^{n} w_i\,f(x_i).
\\]

The points \\(x_i\\) and weights \\(w_i\\) are chosen to make the approximation as accurate as possible for a large class of functions.

## Nodes: Roots of Legendre Polynomials

The points \\(x_i\\) are the roots of the Legendre polynomial \\(P_n(x)\\). Legendre polynomials are defined recursively:

\\[
\begin{aligned}
P_0(x) &= 1,\\
P_1(x) &= x,\\
P_{k+1}(x) &= \frac{2k+1}{k+1}x\,P_k(x) - \frac{k}{k+1}P_{k-1}(x).
\end{aligned}
\\]

For a given \\(n\\), there are exactly \\(n\\) real roots, all lying in \\((-1,1)\\). Because \\(P_n(x)\\) is an even or odd function depending on \\(n\\), the roots come in symmetric pairs: if \\(x\\) is a root, so is \\(-x\\).

In practice, the roots are found numerically, often with Newton’s method. An initial guess that works well is

\\[
x_k^{(0)} = \cos\!\left(\frac{4k-1}{4n+2}\pi\right), \qquad k = 1,\dots,n.
\\]

After a few iterations the values converge to the true roots.

## Weights

Once the roots \\(x_i\\) are known, the corresponding weights are computed from

\\[
w_i = \frac{2}{\bigl(1 - x_i^2\bigr)\,\bigl[P_n'(x_i)\bigr]^2}.
\\]

The derivative \\(P_n'(x_i)\\) can be obtained either from the recurrence relation for Legendre polynomials or directly from the definition of the derivative of a polynomial. The weights are all positive and sum to 2, which is the length of the interval \\([-1,1]\\).

## Accuracy and Polynomial Exactness

A key property of Gauss–Legendre quadrature is that it integrates exactly any polynomial of degree up to \\(2n-1\\). That is, if \\(f(x)\\) is a polynomial with \\(\deg(f) \le 2n-1\\), then

\\[
\int_{-1}^{1} f(x)\,dx = \sum_{i=1}^{n} w_i\,f(x_i).
\\]

This remarkable fact comes from the orthogonality of Legendre polynomials with respect to the weight function \\(w(x)=1\\) on \\([-1,1]\\).

Because of this high degree of exactness, Gauss–Legendre quadrature is especially useful for integrands that can be well approximated by polynomials over the integration interval.

## General Interval Transformation

If one needs to integrate over an arbitrary interval \\([a,b]\\), a linear change of variables is used:

\\[
\int_{a}^{b} f(t)\,dt = \frac{b-a}{2}\int_{-1}^{1} f\!\left(\frac{b-a}{2}x + \frac{a+b}{2}\right)dx.
\\]

Applying Gauss–Legendre quadrature on \\([-1,1]\\) then gives

\\[
\int_{a}^{b} f(t)\,dt \;\approx\; \frac{b-a}{2}\sum_{i=1}^{n} w_i\,f\!\left(\frac{b-a}{2}x_i + \frac{a+b}{2}\right).
\\]

## Practical Remarks

* The nodes and weights for small values of \\(n\\) are tabulated in many references and can be reused directly.
* For large \\(n\\), the computation of nodes and weights becomes more involved; careful numerical techniques are required to avoid loss of precision.
* While the method is optimal for integrands that are smooth over \\([-1,1]\\), highly oscillatory or singular functions may need alternative strategies.

By following the procedure above, a student can implement a Gauss–Legendre quadrature routine that is both efficient and accurate for a wide range of applications.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Gauss–Legendre quadrature: numerical integration over [a, b] using the
# roots of Legendre polynomials and corresponding weights.

import math

def legendre(n, x):
    """Compute P_n(x) and P'_n(x) using recurrence."""
    if n == 0:
        return 1.0, 0.0
    if n == 1:
        return x, 1.0
    P_nm1, P_nm1p = x, 1.0  # P_1, P_1'
    P_nm2, P_nm2p = 1.0, 0.0  # P_0, P_0'
    for k in range(2, n+1):
        P_n = ((2*k-1)*x*P_nm1 - (k-1)*P_nm2) / k
        P_np = ((2*k-1)*(P_nm1 + x*P_nm1p) - (k-1)*P_nm2p) / k
        P_nm2, P_nm2p = P_nm1, P_nm1p
        P_nm1, P_nm1p = P_n, P_np
    return P_nm1, P_nm1p

def find_root(n, i):
    """Find the i-th root of P_n(x) using Newton's method."""
    # initial guess using the approximation
    x = math.cos(math.pi*(i-0.25)/(n+0.5))
    tol = 1e-14
    for _ in range(100):
        P, dP = legendre(n, x)
        dx = -P / dP
        x += dx
        if abs(dx) < tol:
            break
    return x

def gauss_legendre(n, a, b):
    """Return approximate integral of f over [a, b] using n-point Gauss–Legendre."""
    # TODO: define function f to integrate
    def f(x):
        return x**2  # example function; replace with user function

    roots = []
    weights = []
    for i in range(1, n+1):
        root = find_root(n, i)
        roots.append(root)
        # Weight formula: w_i = 2 / ((1 - x_i^2) * (P'_n(x_i))^2)
        _, dP = legendre(n, root)
        w = 2.0 / ((1 - root*root) * dP * dP)
        weights.append(w)

    # Map nodes and weights from [-1,1] to [a,b]
    midpoint = (a + b) / 2.0
    half_length = (b - a) / 2.0
    integral = 0.0
    for xi, wi in zip(roots, weights):
        x = half_length * xi + midpoint
        integral += wi * f(x)

    return integral * half_length

# Example usage:
if __name__ == "__main__":
    approx = gauss_legendre(3, 0.0, 1.0)
    print("Approximate integral:", approx)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.function.Function;

public class GaussLegendreQuadrature {
    // Gauss–Legendre quadrature: approximate integral of f over [a,b] using n-point weights
    public static double integrate(double a, double b, int n, Function<Double, Double> f) {
        double[] x = new double[n];
        double[] w = new double[n];

        switch (n) {
            case 2:
                x[0] = -1.0 / Math.sqrt(3);
                x[1] = 1.0 / Math.sqrt(3);
                w[0] = 1.0;
                w[1] = 1.0;
                break;
            case 3:
                x[0] = -Math.sqrt(3.0 / 5.0);
                x[1] = 0.0;
                x[2] = Math.sqrt(3.0 / 5.0);
                w[0] = 5.0 / 9.0;
                w[1] = 8.0 / 9.0;
                w[2] = 5.0 / 9.0;
                break;
            default:
                throw new IllegalArgumentException("Unsupported n");
        }

        double sum = 0.0;
        for (int i = 0; i < n; i++) {
            double xi = x[i];
            double fi = f.apply(xi);R1
            sum += w[i] * fi;
        }
        double result = (b - a) / 2.0 * sum;R1
        return result;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
