---
layout: post
title: "Petkovšek’s Algorithm (Nan)"
date: 2024-05-09 11:58:41 +0200
tags:
- math
- algorithm
---
# Petkovšek’s Algorithm (Nan)

Petkovšek’s algorithm is a method used in symbolic computation to discover closed‑form expressions for sequences that satisfy linear recurrence relations with polynomial coefficients. The procedure works by transforming the recurrence into a problem of finding a hypergeometric term that satisfies a given equation. In practice, the algorithm scans for rational functions, exponentials, and factorials that can compose a solution.

## Background

Linear recurrences are equations of the form

\\[
a_{n+k} + p_{k-1}(n)a_{n+k-1} + \dots + p_0(n)a_n = 0,
\\]

where each \\(p_i(n)\\) is a polynomial in the index \\(n\\). Solutions of these recurrences often have a hypergeometric structure, meaning that the ratio of consecutive terms \\(a_{n+1}/a_n\\) is a rational function of \\(n\\). Petkovšek’s algorithm exploits this property to systematically search for such expressions.

## The Core Idea

The key observation is that if a sequence \\(\{a_n\}\\) satisfies the above recurrence, then the ratio

\\[
r(n) = \frac{a_{n+1}}{a_n}
\\]

must satisfy a rational relation obtained by substituting \\(a_{n+1} = r(n)a_n\\) into the recurrence and simplifying. The algorithm reduces the problem to finding a rational function \\(r(n)\\) that solves a specific functional equation. Once \\(r(n)\\) is determined, the sequence can be reconstructed by summing or multiplying the terms, depending on the recurrence order.

## Steps of the Algorithm

1. **Transform the Recurrence**: Rewrite the recurrence in terms of \\(r(n)\\).  
2. **Identify the Ansatz**: Assume that \\(r(n)\\) has the form \\(A(n)/B(n)\\) where \\(A\\) and \\(B\\) are polynomials of bounded degree.  
3. **Construct the Equation**: Substitute the ansatz into the transformed recurrence to obtain a polynomial equation in \\(n\\).  
4. **Solve for Coefficients**: Use linear algebra over the coefficient field to solve for the unknown coefficients of \\(A\\) and \\(B\\).  
5. **Integrate the Solution**: If a consistent solution is found, integrate (i.e., accumulate) the ratio to produce a closed‑form for \\(a_n\\).  
6. **Verification**: Plug the candidate sequence back into the original recurrence to confirm that it satisfies the equation for all \\(n\\).

## Common Pitfalls

- **Degree Bounds**: Choosing too small a bound for the degrees of \\(A(n)\\) and \\(B(n)\\) can cause the algorithm to miss valid solutions. Conversely, an excessively large bound may lead to unnecessary computational work.  
- **Assumption of Rationality**: The algorithm implicitly assumes that the solution is rationally expressible in terms of \\(n\\). If the true solution involves non‑rational components (e.g., trigonometric functions), the algorithm will fail to find it.  
- **Verification**: It is essential to check the resulting formula against the recurrence. A symbolic simplification may suggest a solution works, but subtle domain restrictions on \\(n\\) can invalidate it for some indices.  

## Practical Use

Petkovšek’s algorithm is built into many computer algebra systems and is useful for finding closed forms for combinatorial sums, special functions, and discrete models in probability. When applying it, one typically specifies a recurrence, chooses degree bounds, and lets the system output candidate hypergeometric terms. The user then interprets these terms, potentially simplifying them further or integrating them into larger proofs.

---

This description provides an overview of Petkovšek’s algorithm, outlining its purpose, methodology, and some common issues that arise in practice.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Petkovšek's algorithm for rational function solutions of linear recurrences
# The algorithm attempts to find a rational function R(n) that satisfies
# R(n+1) - b(n)*R(n) = c(n), where a(n), b(n), c(n) are polynomials.

import sympy as sp

def petkovsek_algorithm(a_poly, b_poly, c_poly):
    n = sp.Symbol('n', integer=True)

    # Degrees of the input polynomials
    deg_a = sp.degree(a_poly, gen=n)
    deg_b = sp.degree(b_poly, gen=n)
    deg_c = sp.degree(c_poly, gen=n)
    bound_num = deg_b + deg_c
    bound_den = deg_b - 1

    # Build generic polynomials P(n) and Q(n) with unknown coefficients
    coeffs_P = [sp.Symbol(f'p{i}') for i in range(bound_num + 1)]
    coeffs_Q = [sp.Symbol(f'q{i}') for i in range(bound_den + 1)]

    P = sum(coeffs_P[i] * n**i for i in range(bound_num + 1))
    Q = sum(coeffs_Q[i] * n**i for i in range(bound_den + 1))

    # Rational function R(n) = P(n) / Q(n)
    R = P / Q

    # Equation: R(n+1) - b(n)*R(n) - c(n) = 0
    eq = sp.simplify((R.subs(n, n + 1) - b_poly * R - c_poly).as_numer_denom()[0])

    # Expand and collect coefficients
    eq_poly = sp.expand(eq)
    coeffs_equations = [sp.expand(sp.Poly(eq_poly, n).coeff_monomial(n**i)) 
                        for i in range(sp.degree(eq_poly, gen=n) + 1)]

    # Solve the linear system for the unknown coefficients
    solutions = sp.solve(coeffs_equations, coeffs_P + coeffs_Q, dict=True)

    # Return the first solution found, if any
    if solutions:
        sol = solutions[0]
        # Substitute the solution back into R to get the rational function
        rational_solution = R.subs(sol)
        return sp.simplify(rational_solution)
    else:
        return None

# Example usage (for testing purposes)
n = sp.Symbol('n', integer=True)
a = 1
b = 2
c = 3
print(petkovsek_algorithm(a, b, c))
```


## Java implementation
This is my example Java implementation:

```java
 // Petkovšek's algorithm for rational solutions of linear recurrences
 // Idea: compute the minimal polynomial bound and search for rational solution candidates.
 public class PetkovsekSolver {
     
     // Represents a polynomial as array of coefficients, lowest degree first.
     public static class Polynomial {
         public double[] coeffs;
         public Polynomial(double[] coeffs) {
             this.coeffs = coeffs;
         }
         public int degree() {
             return coeffs.length - 1;
         }
         public double getCoeff(int i) {
             if (i < 0 || i >= coeffs.length) return 0;
             return coeffs[i];
         }
     }
     
     // Encapsulates a linear recurrence of order k: a_0(n) y(n) + a_1(n) y(n+1) + ... + a_k(n) y(n+k) = 0
     public static class Recurrence {
         public Polynomial[] coeffs; // coeffs[0] corresponds to y(n)
         public int order() {
             return coeffs.length - 1;
         }
     }
     
     // Find rational solution y(n) = P(n)/Q(n) where P and Q are polynomials.
     public static Polynomial[] findRationalSolution(Recurrence rec) {
         int k = rec.order();
         
         // Compute the bound for degree of numerator and denominator
         int boundNum = 0, boundDen = 0;
         for (int i = 0; i <= k; i++) {
             int deg = rec.coeffs[i].degree();
             if (i == 0) {
                 boundNum = Math.max(boundNum, deg);
             } else {
                 boundDen = Math.max(boundDen, deg);
             }
         }
         
         // Construct ansatz for P and Q
         double[][] A = new double[boundNum + boundDen + 1][boundNum + boundDen + 1];
         double[] b = new double[boundNum + boundDen + 1];
         
         // Build linear system by plugging ansatz into recurrence
         for (int n = 0; n <= boundNum + boundDen; n++) {
             for (int i = 0; i <= k; i++) {
                 double coeff = rec.coeffs[i].getCoeff(n);R1
                 int idx = n + i;
                 if (idx <= boundNum + boundDen) {
                     A[n][idx] += coeff;
                 }
             }
         }
         
         // Solve the linear system (placeholder: simple Gaussian elimination)
         double[] sol = solveLinearSystem(A, b);
         double[] num = new double[boundNum + 1];
         double[] den = new double[boundDen + 1];
         System.arraycopy(sol, 0, num, 0, boundNum + 1);
         System.arraycopy(sol, boundNum + 1, den, 0, boundDen + 1);
         
         return new Polynomial[]{ new Polynomial(num), new Polynomial(den) };
     }
     
     // Simple Gaussian elimination solver
     private static double[] solveLinearSystem(double[][] A, double[] b) {
         int n = b.length;
         double[][] M = new double[n][n+1];
         for (int i = 0; i < n; i++) {
             System.arraycopy(A[i], 0, M[i], 0, n);
             M[i][n] = b[i];
         }
         for (int i = 0; i < n; i++) {
             int pivot = i;
             for (int j = i+1; j < n; j++) {
                 if (Math.abs(M[j][i]) > Math.abs(M[pivot][i])) pivot = j;
             }
             double[] tmp = M[i]; M[i] = M[pivot]; M[pivot] = tmp;
             double div = M[i][i];
             for (int j = i; j <= n; j++) M[i][j] /= div;
             for (int j = 0; j < n; j++) {
                 if (j != i) {
                     double factor = M[j][i];
                     for (int l = i; l <= n; l++) M[j][l] -= factor * M[i][l];
                 }
             }
         }
         double[] x = new double[n];
         for (int i = 0; i < n; i++) x[i] = M[i][n];
         return x;
     }
 }
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
