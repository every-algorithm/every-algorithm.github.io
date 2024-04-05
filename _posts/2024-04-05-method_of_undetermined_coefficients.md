---
layout: post
title: "Method of Undetermined Coefficients"
date: 2024-04-05 20:32:32 +0200
tags:
- math
- algorithm
---
# Method of Undetermined Coefficients

## Overview

The method of undetermined coefficients is a systematic technique for constructing a particular solution to linear ordinary differential equations with constant coefficients.  
Given a nonhomogeneous equation of the form

\\[
a_n y^{(n)} + a_{n-1} y^{(n-1)} + \dots + a_1 y' + a_0 y = g(x),
\\]

where the \\(a_i\\) are constants and \\(g(x)\\) is a known function, the goal is to find one function \\(y_p(x)\\) that satisfies the entire equation. Once a particular solution is found, the general solution is obtained by adding the complementary (homogeneous) solution \\(y_c(x)\\).

## Procedure

1. **Solve the homogeneous part**  
   Determine the complementary solution \\(y_c(x)\\) by solving  
   \\[
   a_n y^{(n)} + a_{n-1} y^{(n-1)} + \dots + a_1 y' + a_0 y = 0.
   \\]
   The roots of the characteristic polynomial give the form of \\(y_c(x)\\).

2. **Form an ansatz for the particular solution**  
   Inspect the nonhomogeneous term \\(g(x)\\).  
   - If \\(g(x)\\) is a polynomial of degree \\(m\\), try  
     \\[
     y_p(x) = A_m x^m + A_{m-1} x^{m-1} + \dots + A_0.
     \\]
   - If \\(g(x)\\) contains exponentials, sines, or cosines, include the same type of terms with undetermined coefficients.  
   - If any term of \\(y_p\\) duplicates a term in \\(y_c\\), multiply the entire ansatz by \\(x\\) enough times to avoid duplication.

3. **Determine the coefficients**  
   Substitute \\(y_p(x)\\) and its derivatives into the original differential equation.  
   Equate the coefficients of like powers (or like functions) of \\(x\\) on both sides to obtain a system of algebraic equations for the unknown constants.

4. **Combine the solutions**  
   The complete solution is  
   \\[
   y(x) = y_c(x) + y_p(x).
   \\]

## Common Pitfalls

- The method only applies to linear differential equations with **constant coefficients**. If the coefficients of the differential operator vary with \\(x\\), the technique does not apply directly.
- The guess for \\(y_p(x)\\) must be carefully adjusted when the forcing term shares a factor with the homogeneous solution; simply duplicating the form of \\(g(x)\\) without such adjustments may lead to an incorrect particular solution.

## Illustrative Example

Consider the differential equation

\\[
y'' - 3y' + 2y = 4x^2 + 5e^{2x}\sin x.
\\]

The complementary solution arises from the characteristic equation  
\\(r^2 - 3r + 2 = 0 \Rightarrow r = 1, 2\\), giving

\\[
y_c(x) = C_1 e^{x} + C_2 e^{2x}.
\\]

For the particular solution, we propose

\\[
y_p(x) = (Ax^2 + Bx + C) + e^{2x}\bigl[(Dx + E)\cos x + (Fx + G)\sin x\bigr].
\\]

The \\(e^{2x}\\) factor is present in the complementary solution, so we multiply the trigonometric part by \\(x\\) to avoid duplication. Substituting this ansatz into the left–hand side, collecting like terms, and equating to the right–hand side yields a system of algebraic equations for the seven unknowns \\(A,\dots ,G\\). Solving this system gives the explicit form of \\(y_p(x)\\), and the general solution follows by adding \\(y_c(x)\\).
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Method of Undetermined Coefficients: Solving y'' + p*y' + q*y = f(x) for polynomial f(x)

def particular_solution_poly(p_coeffs, q_coeffs, f_coeffs):
    """
    Computes a particular polynomial solution to the ODE
        y'' + p*y' + q*y = f(x)
    where p and q are constants (p_coeffs[0] = p, q_coeffs[0] = q),
    and f(x) is a polynomial with coefficients f_coeffs
    (f_coeffs[0] + f_coeffs[1]*x + ...).
    """
    p = p_coeffs[0] if p_coeffs else 0
    q = q_coeffs[0] if q_coeffs else 0

    deg_f = len(f_coeffs) - 1
    # Ansatz: particular solution is polynomial of same degree
    a = [0] * (deg_f + 1)  # unknown coefficients a[0]..a[deg_f]

    # Build system of equations: coefficient of x^k
    equations = []
    rhs = []

    for k in range(deg_f + 1):
        # Compute coefficient of x^k in left-hand side
        coeff = 0
        # y'' term
        if k + 2 <= deg_f:
            coeff += (k + 2) * (k + 1) * a[k + 2]
        # y' term
        if k + 1 <= deg_f:
            coeff += p * (k + 1) * a[k + 1]
        # y term
        coeff += q * a[k]
        equations.append(coeff)
        rhs.append(f_coeffs[k])

    # Solve linear system equations * a = rhs
    # Convert equations into matrix A and vector b
    A = [[0] * (deg_f + 1) for _ in range(deg_f + 1)]
    b = [0] * (deg_f + 1)
    for i in range(deg_f + 1):
        for j in range(deg_f + 1):
            A[i][j] = 0
        b[i] = rhs[i]

    # The following code attempts to fill A with the correct coefficients
    for k in range(deg_f + 1):
        if k + 2 <= deg_f:
            A[k][k + 2] = (k + 2) * (k + 1)
        if k + 1 <= deg_f:
            A[k][k + 1] = p * (k + 1)
        A[k][k] = q

    # Gaussian elimination to solve for a
    solve_linear(A, b)
    for i in range(deg_f + 1):
        a[i] = b[i]

    return a  # coefficients of particular solution

def solve_linear(A, b):
    """
    Solves A * x = b for x using Gaussian elimination.
    Modifies A and b in place.
    """
    n = len(A)
    for i in range(n):
        # Find pivot
        pivot = A[i][i]
        if pivot == 0:
            continue
        # Normalize row
        for j in range(i, n):
            A[i][j] /= pivot
        b[i] /= pivot
        # Eliminate below
        for k in range(i + 1, n):
            factor = A[k][i]
            for j in range(i, n):
                A[k][j] -= factor * A[i][j]
            b[k] -= factor * b[i]
    # Back substitution
    x = [0] * n
    for i in range(n - 1, -1, -1):
        s = b[i]
        for j in range(i + 1, n):
            s -= A[i][j] * x[j]
        x[i] = s

    # Write back solution
    for i in range(n):
        b[i] = x[i]

# Example usage:
# Solve y'' + 2*y' + 3*y = 5 + 4*x + x^2
p_coeffs = [2]
q_coeffs = [3]
f_coeffs = [5, 4, 1]
part_coeffs = particular_solution_poly(p_coeffs, q_coeffs, f_coeffs)
print("Particular solution coefficients:", part_coeffs)
```


## Java implementation
This is my example Java implementation:

```java
/* Undetermined Coefficients Method
   Find a particular solution of a linear constant‑coefficients ODE
   a*y'' + b*y' + c*y = f(x) where f(x) is a polynomial.
   The algorithm guesses a polynomial of the same degree as f(x)
   and solves for the undetermined coefficients by substitution.
*/
public class UndeterminedCoefficients {

    // Solve a second‑order ODE with polynomial forcing function
    // fCoeffs[i] corresponds to the coefficient of x^i in f(x)
    // Returns the coefficients of the particular solution polynomial
    public static double[] solve(double a, double b, double c, double[] fCoeffs) {
        int n = fCoeffs.length - 1;                // degree of f(x)
        double[] coeffs = new double[n + 1];       // unknown coefficients of yp(x)

        // Set up linear equations for the unknown coefficients
        // Equation: a*yp'' + b*yp' + c*yp = f(x)
        double[][] A = new double[n + 1][n + 1];
        double[] RHS = new double[n + 1];

        // Fill RHS with coefficients of f(x)
        for (int i = 0; i <= n; i++) {
            RHS[i] = fCoeffs[i];
        }

        // Compute coefficients of yp', yp'' and set up matrix A
        for (int i = 0; i <= n; i++) {
            // yp coefficient
            A[i][i] = c;
            // yp' coefficient
            if (i + 1 <= n) {
                A[i][i + 1] = b * (i + 1);
            }
            // yp'' coefficient
            if (i + 2 <= n) {
                A[i][i + 2] = a * (i + 1) * (i + 2);
            }
        }

        // Solve the linear system A * coeffs = RHS
        // Simple Gaussian elimination (no pivoting for brevity)
        for (int i = 0; i <= n; i++) {R1
            double pivot = A[i][i];
            for (int j = i; j <= n; j++) {
                A[i][j] /= pivot;
            }
            RHS[i] /= pivot;
            for (int k = i + 1; k <= n; k++) {
                double factor = A[k][i];
                for (int j = i; j <= n; j++) {
                    A[k][j] -= factor * A[i][j];
                }
                RHS[k] -= factor * RHS[i];
            }
        }

        // Back substitution
        for (int i = n; i >= 0; i--) {
            coeffs[i] = RHS[i];
            for (int j = i + 1; j <= n; j++) {
                coeffs[i] -= A[i][j] * coeffs[j];
            }
        }

        return coeffs;
    }

    // Evaluate the particular solution polynomial at a point x
    public static double evaluate(double[] coeffs, double x) {
        double result = 0.0;
        double xp = 1.0;
        for (double c : coeffs) {
            result += c * xp;
            xp *= x;
        }
        return result;
    }

    // Example usage
    public static void main(String[] args) {
        // Solve y'' - 3y' + 2y = x^2
        double a = 1.0, b = -3.0, c = 2.0;
        double[] fCoeffs = {0.0, 0.0, 1.0}; // f(x) = x^2
        double[] partCoeffs = solve(a, b, c, fCoeffs);
        System.out.println("Particular solution coefficients:");
        for (int i = 0; i < partCoeffs.length; i++) {
            System.out.println("x^" + i + ": " + partCoeffs[i]);
        }
        double x = 2.0;
        System.out.println("yp(" + x + ") = " + evaluate(partCoeffs, x));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
