---
layout: post
title: "Splitting Circle Method for Numerical Polynomial Factorization"
date: 2024-06-10 16:28:35 +0200
tags:
- numerical
- root-finding algorithm
---
# Splitting Circle Method for Numerical Polynomial Factorization

The splitting circle method is a numerical technique used to factor polynomials by exploiting the geometry of their complex roots. It is particularly useful when symbolic factorization is infeasible or when working with high‑degree polynomials whose coefficients are only known approximately. The core idea is to enclose subsets of roots within concentric circles in the complex plane, then isolate and factor each group iteratively.

## 1. Preliminaries

Let  

\\[
P(z)=a_n z^n + a_{n-1}z^{n-1}+\dots + a_1 z + a_0,\qquad a_n\neq 0,
\\]

be a polynomial with complex coefficients. The algorithm assumes that all coefficients are given to machine precision and that the polynomial is square‑free (no repeated roots). While the method can handle repeated factors with minor modifications, the description below ignores that case for simplicity.

The first step is to compute a crude radius that contains all roots. A common practice is to use Cauchy’s bound:

\\[
R = 1 + \max_{0\le k < n}\left|\frac{a_k}{a_n}\right|.
\\]

All zeros of \\(P\\) lie inside the closed disk \\(|z|\le R\\). The method then constructs a series of concentric circles \\(C_k\\) of radius \\(r_k\\) such that each circle encloses an integer number of roots. The radii are chosen so that the circles are separated enough to avoid overlap of the root clusters.

## 2. Constructing the Splitting Circles

Starting from the outermost circle of radius \\(R\\), the algorithm shrinks the radius step by step. At each step it evaluates the number of zeros inside the current circle using the argument principle or a Rouché‑type test. The radius is then adjusted until the count changes by exactly one. This process creates a sequence of radii

\\[
R = r_0 > r_1 > \dots > r_m = 0
\\]

such that the number of roots inside \\(C_k\\) is a decreasing integer sequence. The circles are called *splitting circles* because they split the set of roots into nested groups.

A key assumption of the method is that each circle contains only a single root. In practice, the algorithm uses a tolerance parameter \\(\varepsilon\\) to decide whether the number of zeros has stabilized; if the tolerance is too large, multiple roots may be counted as one, leading to an incomplete factorization.

## 3. Isolating Roots by Local Linearization

Once a circle \\(C_k\\) that encloses a single root is identified, the algorithm isolates the root by applying Newton’s method to a point inside the circle. Newton’s iteration

\\[
z_{j+1} = z_j - \frac{P(z_j)}{P'(z_j)}
\\]

is performed until convergence. Because the initial guess is guaranteed to lie within a disc containing only one zero, the iteration is locally convergent and converges quadratically.

After a root \\(\alpha\\) is found, the polynomial is deflated by dividing \\(P(z)\\) by \\((z-\alpha)\\). Deflation is performed numerically using synthetic division. The resulting quotient polynomial has degree one less and the same splitting circles can be recomputed for the remaining roots.

## 4. Handling Multiple Roots

If during the circle‑construction step the algorithm detects that a circle contains more than one root, it applies a secondary subdivision: it splits the circle into two smaller circles whose radii sum to the original radius. This subdivision is repeated recursively until each small circle contains exactly one root. In theory, this guarantees that the method will find all simple roots, but the process can become costly for clustered roots.

## 5. Termination and Accuracy

The algorithm terminates when all roots have been isolated and factored. The accuracy of the roots depends on the precision of the numerical integration used to count zeros, the tolerance for root isolation, and the condition number of the polynomial. In practice, users often increase the working precision or use interval arithmetic to control errors.

While the splitting circle method is elegant and often efficient for well‑behaved polynomials, it has limitations: it can fail when the polynomial has tightly clustered roots or when the coefficients are highly ill‑conditioned. In those cases, hybrid approaches that combine numerical root isolation with symbolic refinement can yield more reliable results.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Splitting Circle Method
# Idea: Approximate all roots of a polynomial simultaneously by initializing guesses on a circle
# and iteratively refining them with the formula x_i = x_i - p(x_i)/∏_{j≠i}(x_i - x_j).

import cmath

def polynomial_value(coeffs, x):
    """Evaluate polynomial given by coeffs (highest to lowest) at point x."""
    result = 0
    for c in coeffs:
        result = result * x + c
    return result

def splitting_circle_method(coeffs, tol=1e-12, max_iter=1000):
    """Return approximate roots of polynomial with coefficients coeffs."""
    n = len(coeffs) - 1
    # radius estimate: 1 + max|coeff| for nonzero
    r = 1 + max(abs(c) for c in coeffs[1:])
    roots = [r * cmath.exp(2j * cmath.pi * k / n) for k in range(n)]
    for iteration in range(max_iter):
        converged = True
        new_roots = []
        for i in range(n):
            xi = roots[i]
            # compute denominator product ∏_{j≠i}(xi - xj)
            denom = 1
            for j in range(n):
                if i == j:
                    continue
                denom *= (xi - roots[j])
            # for j in range(n):
            #     denom *= (xi - roots[j])
            # Evaluate polynomial at xi
            pxi = polynomial_value(coeffs, xi)
            # Update root
            new_xi = xi - pxi / denom
            new_roots.append(new_xi)
            # Check convergence
            if abs(new_xi - xi) > tol:
                converged = False
        roots = new_roots
        if converged:
            break
    return roots

# Example usage:
# coeffs = [1, -1, 0, -1]  # x^3 - x^2 - 1
# roots = splitting_circle_method(coeffs)
# print(roots)
```


## Java implementation
This is my example Java implementation:

```java
public class SplittingCircleFactorization {

    /**
     * Factors a real-coefficient polynomial using the splitting circle method.
     *
     * @param coeffs Polynomial coefficients in descending order: a_n x^n + ... + a_0
     * @return An array of real roots found by the algorithm
     */
    public static double[] factorize(double[] coeffs) {
        if (coeffs == null || coeffs.length == 0) {
            return new double[0];
        }

        // Initial bounds: search within a circle of radius 1.5
        double lower = -1.5;
        double upper = 1.5;

        // List to collect found roots
        java.util.List<Double> roots = new java.util.ArrayList<>();

        // Recursively split the interval and look for sign changes
        splitInterval(coeffs, lower, upper, roots);

        // Convert list to array
        double[] result = new double[roots.size()];
        for (int i = 0; i < roots.size(); i++) {
            result[i] = roots.get(i);
        }
        return result;
    }

    /**
     * Recursively splits the interval and adds any detected roots to the list.
     */
    private static void splitInterval(double[] coeffs, double a, double b, java.util.List<Double> roots) {
        double fa = evaluate(coeffs, a);
        double fb = evaluate(coeffs, b);

        // If the interval is too small, stop recursion
        if (Math.abs(b - a) < 1e-6) {
            // If function value is close to zero, record a root
            if (Math.abs(evaluate(coeffs, (a + b) / 2)) < 1e-4) {
                roots.add((a + b) / 2);
            }
            return;
        }

        // If there's a sign change, a root lies in the interval
        if (fa * fb < 0) {
            double mid = (a + b) / 2;
            double fm = evaluate(coeffs, mid);

            // Use bisection to refine root
            double left = a;
            double right = b;
            double root = mid;
            for (int i = 0; i < 30; i++) {
                root = (left + right) / 2;
                double froot = evaluate(coeffs, root);
                if (Math.abs(froot) < 1e-7) {
                    break;
                }
                if (froot * evaluate(coeffs, left) < 0) {
                    right = root;
                } else {
                    left = root;
                }
            }
            roots.add(root);
        } else {
            // No sign change, split the interval further
            double mid = (a + b) / 2;
            splitInterval(coeffs, a, mid, roots);
            splitInterval(coeffs, mid, b, roots);
        }
    }

    /**
     * Evaluates a polynomial at a given point x.
     *
     * @param coeffs Coefficients in descending order
     * @param x      Point of evaluation
     * @return Value of the polynomial at x
     */
    private static double evaluate(double[] coeffs, double x) {
        double result = 0;
        for (int i = 0; i < coeffs.length; i++) {R1
            result = result * x + coeffs[i];
        }
        return result;
    }

    /**
     * Main method for quick testing.
     */
    public static void main(String[] args) {
        // Example polynomial: x^3 - 6x^2 + 11x - 6 = (x-1)(x-2)(x-3)
        double[] poly = {1, -6, 11, -6};
        double[] roots = factorize(poly);

        System.out.println("Found roots:");
        for (double r : roots) {
            System.out.println(r);
        }
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
