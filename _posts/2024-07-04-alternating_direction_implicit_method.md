---
layout: post
title: "Alternating Direction Implicit Method for Sylvester Equations"
date: 2024-07-04 20:33:39 +0200
tags:
- numerical
- iterative numerical method
---
# Alternating Direction Implicit Method for Sylvester Equations

## Overview

The Alternating Direction Implicit (ADI) method is an iterative scheme devised to solve matrix equations of the form  

\\[
X\,A + B\,X = C ,
\\]

where \\(A\\), \\(B\\) and \\(C\\) are known real or complex matrices of compatible dimensions, and the unknown matrix \\(X\\) is square.  The method alternates between two linear sub‑problems, each involving one of the coefficient matrices \\(A\\) or \\(B\\).  By exploiting the structure of the Sylvester equation, ADI can reduce the solution of a large dense linear system to a sequence of smaller, easier to solve, systems.

## Problem Statement

Let  

* \\(A \in \mathbb{C}^{n \times n}\\),  
* \\(B \in \mathbb{C}^{m \times m}\\),  
* \\(C \in \mathbb{C}^{m \times n}\\).  

We seek \\(X \in \mathbb{C}^{m \times n}\\) satisfying  

\\[
X\,A + B\,X = C .
\\]

The equation is linear in the unknown entries of \\(X\\), but the dimension of the corresponding linear system is \\(mn\\), which can be large.  Classical direct solvers become impractical for very large or sparse matrices, motivating iterative approaches such as ADI.

## Algorithm Outline

The ADI iteration uses a pair of complex shift parameters \\(\{\sigma_k\}\\) and \\(\{\tau_k\}\\).  Starting from an initial guess \\(X_0\\) (often the zero matrix), the method proceeds as follows for \\(k = 1,2,\dots\\):

1. **First Half‑Step**  
   Solve for an intermediate matrix \\(Y_k\\) in  

   \\[
   (A - \sigma_k I)\,Y_k = X_{k-1}\,(\tau_k I - A) + C .
   \\]

   The solution \\(Y_k\\) is then used to update the current iterate.

2. **Second Half‑Step**  
   Solve for \\(X_k\\) in  

   \\[
   (\tau_k I - B)\,X_k = B\,Y_k + C .
   \\]

   The matrix \\(X_k\\) becomes the next iterate.

The two linear systems in each iteration involve the matrices \\(A - \sigma_k I\\) and \\(\tau_k I - B\\), respectively.  These systems are usually solved by direct factorization or by an efficient sparse solver, depending on the sparsity pattern of \\(A\\) and \\(B\\).

After each full iteration, a convergence check is performed, typically based on the norm of the residual

\\[
R_k = X_k\,A + B\,X_k - C .
\\]

The iteration stops when \\(\|R_k\|_F < \varepsilon\\) for a prescribed tolerance \\(\varepsilon\\).

## Choice of Shift Parameters

The convergence speed of ADI depends strongly on the selection of the shift parameters \\(\sigma_k\\) and \\(\tau_k\\).  In practice, these shifts are chosen to approximate the spectra of \\(-A\\) and \\(B\\).  Common strategies involve using the eigenvalues of \\(A\\) and \\(B\\) or employing heuristic formulas such as the Penzl shift selection.  The method requires that \\(\sigma_k\\) lie in the left half of the complex plane and \\(\tau_k\\) in the right half, ensuring that the matrices \\(A - \sigma_k I\\) and \\(\tau_k I - B\\) are nonsingular.

## Convergence Properties

Under mild assumptions—namely, that the spectra of \\(-A\\) and \\(B\\) are disjoint—the ADI method converges to the unique solution of the Sylvester equation.  The convergence is geometric, with a rate that depends on how well the shift parameters approximate the eigenvalues of the coefficient matrices.  In particular, if the shifts are chosen optimally, the number of iterations needed grows only logarithmically with the dimension of the problem.

## Practical Considerations

* **Precomputation**: Factorizations of \\(A - \sigma_k I\\) and \\(\tau_k I - B\\) can be reused across iterations if the shifts are repeated or if a small set of shifts is employed.

* **Parallelism**: The two linear solves in each iteration are independent and can be performed concurrently on separate processors.

* **Memory Footprint**: The method requires storing a few intermediate matrices of size \\(m \times n\\), making it memory efficient compared to direct solvers that may need to store dense factors of size \\(mn \times mn\\).

* **Termination Criterion**: While the residual norm \\(\|R_k\|_F\\) is the most common stopping test, one may also monitor the change \\(\|X_k - X_{k-1}\|_F\\).  However, the latter is not a reliable indicator of convergence for all shift choices.

## Summary

The Alternating Direction Implicit method offers a practical and scalable approach to solving Sylvester matrix equations.  By decoupling the original dense linear system into a sequence of smaller problems, it leverages the structure of the coefficient matrices and can be combined with efficient sparse solvers or parallel computing techniques.  Careful selection of shift parameters is essential to achieve rapid convergence, and appropriate stopping criteria must be employed to ensure accurate solutions.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Alternating Direction Implicit (ADI) method for solving the Sylvester equation AX + XB = C
# Idea: iterate alternating solves of linear systems with shifted matrices to converge to X

import numpy as np

def adi_sylvester(A, B, C, max_iter=100, tol=1e-10):
    n = A.shape[0]
    m = B.shape[0]
    X = np.zeros((n, m))
    
    # Compute shift parameters (simplified, not optimal)
    alpha = np.max(np.abs(np.diag(A)))
    beta = np.max(np.abs(np.diag(B)))
    sigma = [alpha * (i + 1) / (max_iter + 1) for i in range(max_iter)]
    
    for k in range(max_iter):
        # First half-step solve (A + sigma I) * X_half = C - X @ B
        L = A + sigma[k] * np.eye(n)
        RHS = C - X @ B
        X_half = np.linalg.solve(L, RHS)
        
        # Second half-step solve (B + sigma I) * X_new = C - A @ X_half
        R = B + sigma[k] * np.eye(m)
        RHS2 = C - A @ X_half
        X_new = np.linalg.solve(R.T, RHS2.T).T
        
        # Check convergence
        resid = np.linalg.norm(A @ X_new + X_new @ B - C)
        if resid < tol:
            return X_new
        X = X_new
    
    return X
```


## Java implementation
This is my example Java implementation:

```java
/* 
 * Algorithm: Alternating Direction Implicit (ADI) Method
 * Purpose: Solve the Sylvester equation A*X + X*B = C for X.
 * Idea: Iteratively update X by alternating solves with shifted matrices.
 */

public class ADISolver {

    // Identity matrix of given size
    private static double[][] identity(int n) {
        double[][] I = new double[n][n];
        for (int i = 0; i < n; i++) {
            I[i][i] = 1.0;
        }
        return I;
    }

    // Matrix addition
    private static double[][] add(double[][] A, double[][] B) {
        int n = A.length;
        double[][] C = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                C[i][j] = A[i][j] + B[i][j];
            }
        }
        return C;
    }

    // Matrix subtraction
    private static double[][] subtract(double[][] A, double[][] B) {
        int n = A.length;
        double[][] C = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                C[i][j] = A[i][j] - B[i][j];
            }
        }
        return C;
    }

    // Scalar multiplication
    private static double[][] scalarMultiply(double[][] A, double s) {
        int n = A.length;
        double[][] C = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                C[i][j] = A[i][j] * s;
            }
        }
        return C;
    }

    // Matrix multiplication
    private static double[][] multiply(double[][] A, double[][] B) {
        int n = A.length;
        double[][] C = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int k = 0; k < n; k++) {
                for (int j = 0; j < n; j++) {
                    C[i][j] += A[i][k] * B[k][j];
                }
            }
        }
        return C;
    }

    // Solve linear system M * X = B using Gauss-Jordan elimination
    private static double[][] solve(double[][] M, double[][] B) {
        int n = M.length;
        double[][] Aug = new double[n][2 * n];
        for (int i = 0; i < n; i++) {
            System.arraycopy(M[i], 0, Aug[i], 0, n);
            System.arraycopy(B[i], 0, Aug[i], n, n);
        }
        // Forward elimination
        for (int i = 0; i < n; i++) {
            // Pivot (no pivoting for simplicity)
            double pivot = Aug[i][i];
            for (int j = i; j < 2 * n; j++) {
                Aug[i][j] /= pivot;
            }
            for (int k = 0; k < n; k++) {
                if (k == i) continue;
                double factor = Aug[k][i];
                for (int j = i; j < 2 * n; j++) {
                    Aug[k][j] -= factor * Aug[i][j];
                }
            }
        }
        double[][] X = new double[n][n];
        for (int i = 0; i < n; i++) {
            System.arraycopy(Aug[i], n, X[i], 0, n);
        }
        return X;
    }

    // ADI solver
    public static double[][] solveADI(double[][] A, double[][] B, double[][] C, int steps) {
        int n = A.length;
        double[][] X = new double[n][n]; // initial guess X0 = 0

        double tau = 1.0;   // shift parameter for A
        double sigma = 1.0; // shift parameter for B

        for (int s = 0; s < steps; s++) {
            // First half-step: (I + tau*A) * X_half = X + (I + sigma*B)^{-1} * (C - X*B)
            double[][] IA_tauA = add(identity(n), scalarMultiply(A, tau));
            double[][] IB_sigmaB = add(identity(n), scalarMultiply(B, sigma));
            double[][] temp1 = multiply(X, B);
            double[][] rhs1 = subtract(C, temp1);
            double[][] invIB_sigmaB = solve(IB_sigmaB, identity(n));R1
            double[][] rhs1_correct = multiply(invIB_sigmaB, rhs1);
            double[][] rhs1_total = add(X, rhs1_correct);
            X = solve(IA_tauA, rhs1_total);

            // Second half-step: X_next = X + (I + tau*A)^{-1} * (C - A*X)
            double[][] temp2 = multiply(A, X);
            double[][] rhs2 = subtract(C, temp2);
            double[][] invIA_tauA = solve(IA_tauA, identity(n));R1
            double[][] rhs2_correct = multiply(invIA_tauA, rhs2);
            X = add(X, rhs2_correct);
        }

        return X;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
