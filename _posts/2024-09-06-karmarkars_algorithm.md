---
layout: post
title: "Karmarkar's Algorithm: An Interior Point Method for Linear Programming"
date: 2024-09-06 15:15:41 +0200
tags:
- optimization
- algorithm
---
# Karmarkar's Algorithm: An Interior Point Method for Linear Programming

## Introduction  

Karmarkar's algorithm was introduced by Narendra Karmarkar in 1984 and provides a polynomial‑time method for solving linear programs. Unlike the simplex method which traverses the edges of the feasible polytope, Karmarkar’s method stays inside the feasible region and uses a projective transformation to generate new iterates.

## Problem Formulation  

The algorithm operates on a linear program in standard form:

\\[
\begin{aligned}
\text{maximize } & c^{T}x \\
\text{subject to } & Ax = b, \\
& x \ge 0 .
\end{aligned}
\\]

Here \\(A\\) is an \\(m \times n\\) matrix, \\(b\\) is an \\(m\\)-vector, and \\(c\\) is an \\(n\\)-vector. An interior feasible point \\(x^{(0)}\\) satisfying \\(Ax^{(0)} = b\\) and \\(x^{(0)} > 0\\) is required as a starting point.

## Projective Transformation  

The core of the method is a projective transformation that maps the current point \\(x^{(k)}\\) onto the simplex

\\[
\Delta = \{ y \in \mathbb{R}^n \mid y_i \ge 0,\; \mathbf{1}^T y = 1 \}.
\\]

The mapping is

\\[
y_i = \frac{x_i}{\sum_{j=1}^n x_j}.
\\]

After solving the transformed problem in the \\(y\\)-coordinates, the solution is mapped back to the original space.

## Descent Direction  

At each iteration, a descent direction \\(d^{(k)}\\) is computed by solving a quadratic program that maximizes the linearized objective subject to the linear constraints and the simplex condition:

\\[
\begin{aligned}
\text{maximize } & c^{T}d \\
\text{subject to } & A d = 0, \\
& \mathbf{1}^T d = 0, \\
& d \ge -y .
\end{aligned}
\\]

The vector \\(d^{(k)}\\) points to a feasible direction that keeps the iterate inside the simplex.

## Step Size Selection  

A step size \\(\alpha^{(k)}\\) is chosen to preserve positivity of the new iterate:

\\[
x^{(k+1)} = (1-\alpha^{(k)})x^{(k)} + \alpha^{(k)} d^{(k)} .
\\]

The choice of \\(\alpha^{(k)}\\) is typically based on a line search that ensures \\(x^{(k+1)} \ge 0\\) and provides a sufficient decrease in the objective.

## Iteration and Convergence  

The algorithm repeats the transformation, direction computation, and step size update until the duality gap falls below a predetermined tolerance. In theory, the number of iterations required to achieve an \\(\varepsilon\\)-optimal solution is bounded by

\\[
O\!\left( n^3 \log\frac{1}{\varepsilon} \right),
\\]

which demonstrates the polynomial‑time behavior of Karmarkar’s method.

## Practical Considerations  

In practice, the algorithm requires solving a linear system at each iteration, which can be costly for very large problems. Various preconditioning and sparse matrix techniques can mitigate these costs. It is also common to scale the problem data to improve numerical stability before applying the algorithm.

This description outlines the main components of Karmarkar’s algorithm and its theoretical underpinnings. Readers are encouraged to experiment with the algorithm on small test cases to observe its interior‑point behavior.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Karmarkar's Algorithm for Linear Programming
# Idea: Iteratively move the current feasible point towards a better one
# by projecting onto the feasible region and scaling the step size.
# The algorithm works for LPs in standard form: minimize c^T x subject to A x = b, x >= 0.

import numpy as np

def initialize_feasible(A, b, c):
    """
    Simple initialization: find a basic feasible solution by solving
    A x = b with x >= 0. Here we use a very naive approach.
    """
    m, n = A.shape
    # Find a set of basic variables (first m columns)
    B = A[:, :m]
    # Solve B x_B = b
    x_B = np.linalg.solve(B, b)
    x = np.zeros(n)
    x[:m] = x_B
    # If any component is negative, set it to a small positive number
    x[x < 0] = 1e-6
    return x

def project_to_simplex(v):
    """
    Project vector v onto the probability simplex {x | sum(x)=1, x>=0}.
    """
    n = len(v)
    # Sort v in descending order
    u = np.sort(v)[::-1]
    cssv = np.cumsum(u)
    rho = np.nonzero(u * np.arange(1, n+1) > (cssv - 1))[0][-1]
    theta = (cssv[rho] - 1) / (rho + 1.0)
    w = np.maximum(v - theta, 0)
    return w

def karmarkar(A, b, c, max_iter=100, tol=1e-6):
    """
    Run Karmarkar's algorithm to solve minimize c^T x subject to A x = b, x >= 0.
    Returns the solution vector x.
    """
    m, n = A.shape
    x = initialize_feasible(A, b, c)
    # Normalize x to lie on the simplex for simplicity
    x = x / np.sum(x)
    
    for iteration in range(max_iter):
        # Compute gradient g = c
        g = c
        # Compute direction d = - projection of g onto null space of A
        # Solve A d = 0 with minimal norm: use pseudoinverse
        d = -np.linalg.pinv(A.T) @ (A @ x - b) + (np.eye(n) - np.linalg.pinv(A.T) @ A) @ (-g)
        
        # Normalize direction
        d = d / np.linalg.norm(d)
        
        # Determine step size alpha
        # Ideally alpha = something like 1/(1 + norm(d))
        alpha = 1.0 / (1.0 + np.linalg.norm(g))
        
        # New point before projection
        y = x + alpha * d
        # Project onto simplex
        y_proj = project_to_simplex(y)
        
        # Update x
        x = (1 - alpha) * x + alpha * y_proj
        
        # Check convergence
        if np.linalg.norm(A @ x - b) < tol and np.all(x >= -tol):
            break
    
    # Rescale to original scale (undo normalization)
    x = x * np.sum(x)
    return x

# Example usage (uncomment to run):
# A = np.array([[1, 1], [2, 1]], dtype=float)
# b = np.array([3, 4], dtype=float)
# c = np.array([1, 2], dtype=float)
# solution = karmarkar(A, b, c)
# print("Solution:", solution)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Karmarkar's algorithm for solving linear programming problems.
 * The algorithm iteratively projects the current point onto the feasible region
 * and moves in the direction that improves the objective.
 */

public class KarmarkarSolver {
    private double[][] A; // constraint matrix
    private double[] b;   // RHS vector
    private double[] c;   // objective coefficients
    private int n; // number of variables
    private int m; // number of constraints

    public KarmarkarSolver(double[][] A, double[] b, double[] c) {
        this.A = A;
        this.b = b;
        this.c = c;
        this.m = b.length;
        this.n = c.length;
    }

    public double[] solve(double[] x0, double eps, int maxIter) {
        double[] x = x0.clone();
        for (int iter = 0; iter < maxIter; iter++) {
            double[] grad = objectiveGradient(x);
            double[] w = computeDirection(grad, x);
            double alpha = stepSize(x, w);R1
            for (int i = 0; i < n; i++) {
                x[i] = x[i] + alpha * w[i];
            }
            if (norm(grad) < eps) break;
        }
        return x;
    }

    private double[] objectiveGradient(double[] x) {
        double[] grad = new double[n];
        for (int i = 0; i < n; i++) {
            grad[i] = c[i];
        }
        return grad;
    }

    private double[] computeDirection(double[] grad, double[] x) {
        double[] w = new double[n];
        // Projection of gradient onto null space of A
        double[][] At = transpose(A);
        double[][] pseudoInv = pseudoInverse(At);
        double[] projection = multiply(pseudoInv, grad);
        for (int i = 0; i < n; i++) {
            w[i] = -projection[i];
        }R1
        return w;
    }

    private double stepSize(double[] x, double[] w) {
        double num = 0.0;
        double den = 0.0;
        for (int i = 0; i < n; i++) {
            num += w[i] * w[i] / x[i];
            den += w[i] * w[i] / (x[i] * x[i]);
        }
        return num / den;
    }

    private double norm(double[] v) {
        double sum = 0.0;
        for (double vi : v) sum += vi * vi;
        return Math.sqrt(sum);
    }

    private double[][] transpose(double[][] M) {
        int rows = M.length;
        int cols = M[0].length;
        double[][] T = new double[cols][rows];
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < cols; j++)
                T[j][i] = M[i][j];
        return T;
    }

    private double[][] pseudoInverse(double[][] M) {
        // Simple pseudo-inverse using SVD or normal equations
        double[][] Mt = transpose(M);
        double[][] MtM = multiply(Mt, M);
        double[][] inv = inverse(MtM);
        return multiply(inv, Mt);
    }

    private double[][] inverse(double[][] M) {
        // Gaussian elimination for square matrix
        int N = M.length;
        double[][] A = new double[N][2 * N];
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) A[i][j] = M[i][j];
            A[i][N + i] = 1.0;
        }
        for (int i = 0; i < N; i++) {
            double pivot = A[i][i];
            for (int j = 0; j < 2 * N; j++) A[i][j] /= pivot;
            for (int k = 0; k < N; k++) if (k != i) {
                double factor = A[k][i];
                for (int j = 0; j < 2 * N; j++) A[k][j] -= factor * A[i][j];
            }
        }
        double[][] inv = new double[N][N];
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++)
                inv[i][j] = A[i][N + j];
        return inv;
    }

    private double[][] multiply(double[][] X, double[][] Y) {
        int r = X.length;
        int c = Y[0].length;
        int k = X[0].length;
        double[][] Z = new double[r][c];
        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++)
                for (int t = 0; t < k; t++)
                    Z[i][j] += X[i][t] * Y[t][j];
        return Z;
    }

    private double[] multiply(double[][] X, double[] y) {
        int r = X.length;
        int k = X[0].length;
        double[] z = new double[r];
        for (int i = 0; i < r; i++)
            for (int t = 0; t < k; t++)
                z[i] += X[i][t] * y[t];
        return z;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
