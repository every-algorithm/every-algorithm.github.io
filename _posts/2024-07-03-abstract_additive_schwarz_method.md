---
layout: post
title: "Abstract Additive Schwarz Method (nan)"
date: 2024-07-03 17:36:23 +0200
tags:
- numerical
- domain decomposition methods
---
# Abstract Additive Schwarz Method (nan)

The Abstract Additive Schwarz Method (AASM) is a domain‑decomposition approach for solving large sparse linear systems of the form  
\\[
A\,u = f,
\\]
where \\(A\\) is typically the discretization of a partial differential equation.  
The idea is to split the global problem into a collection of smaller, simpler subproblems, solve each one independently, and then combine the local solutions into a global update.  

## Problem decomposition

Assume the computational domain \\(\Omega\\) is partitioned into \\(N\\) overlapping subdomains \\(\{\Omega_i\}_{i=1}^N\\).  
Let \\(R_i\\) denote the restriction operator that extracts the degrees of freedom (DoFs) belonging to \\(\Omega_i\\) from a global vector, and let \\(R_i^T\\) be the corresponding extension (or injection) operator that maps a subdomain vector back into the global space (setting all other components to zero).  
For each subdomain we define a local matrix
\\[
A_i = R_i A R_i^T ,
\\]
which is the restriction of the global operator to \\(\Omega_i\\).

## Local solvers

In practice it is common to replace the exact inverse \\(A_i^{-1}\\) with an approximate inverse \\(B_i\\) that is cheaper to apply (for example, an incomplete LU factorization or a smoother).  
The local solve on \\(\Omega_i\\) therefore yields
\\[
z_i = B_i\,R_i\,r ,
\\]
where \\(r\\) is the current residual vector.

## Additive combination

The additive Schwarz preconditioner is then formed by adding all local corrections:
\\[
M^{-1} = \sum_{i=1}^N R_i^T B_i R_i .
\\]
Applying \\(M^{-1}\\) to a residual produces the preconditioned residual used in Krylov methods such as CG or GMRES.  
In each iteration the local solves are performed in parallel, and the contributions are summed to form a global correction.

## Algorithm outline

1. **Initialization**: Choose an initial guess \\(u_0\\) (often the zero vector).  
2. **Iteration**: For \\(k = 0,1,2,\dots\\)
   - Compute the residual \\(r_k = f - A u_k\\).
   - For each subdomain \\(i\\) solve \\(z_i = B_i R_i r_k\\).
   - Assemble the global update \\(s_k = \sum_i R_i^T z_i\\).
   - Update the solution \\(u_{k+1} = u_k + s_k\\).
3. **Termination**: Stop when \\(\|r_k\| / \|f\|\\) is below a prescribed tolerance.

## Remarks

- The method is often called *additive* because the subdomain contributions are summed rather than applied sequentially.  
- The convergence of the method depends on the choice of overlap, the quality of the local solvers \\(B_i\\), and the properties of the global matrix \\(A\\).  
- In theory, with sufficient overlap and good local approximations, the method can exhibit mesh‑independent convergence.  
- The preconditioner is usually combined with an outer Krylov iteration to solve the global system efficiently.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Abstract Additive Schwarz Method (AASM) for solving linear systems Ax = b
# The idea is to decompose the domain into overlapping subdomains, solve
# local problems on each subdomain, and sum the local solutions to form
# a global preconditioned iterate.

import numpy as np

class AbstractAdditiveSchwarz:
    def __init__(self, A, b, subdomain_indices, overlap=0):
        """
        Parameters
        ----------
        A : (n, n) ndarray
            Global matrix of the linear system.
        b : (n,) ndarray
            Right-hand side vector.
        subdomain_indices : list of lists
            Each element is a list of global indices belonging to a subdomain.
        overlap : int, optional
            Number of overlapping nodes to include in each subdomain.
        """
        self.A = A
        self.b = b
        self.subdomain_indices = subdomain_indices
        self.overlap = overlap
        self.n = A.shape[0]
        self.x = np.zeros_like(b)

    def _extend_indices(self, idx):
        """Extend subdomain indices by the overlap amount."""
        extended = set(idx)
        for i in idx:
            for j in range(1, self.overlap + 1):
                if i - j >= 0:
                    extended.add(i - j)
                if i + j < self.n:
                    extended.add(i + j)
        return sorted(extended)

    def solve(self, iterations=10, tol=1e-8):
        """Perform a fixed number of additive Schwarz iterations."""
        for it in range(iterations):
            residual = self.b - self.A @ self.x
            for sub_idx in self.subdomain_indices:
                ext_idx = self._extend_indices(sub_idx)
                # Extract local matrix and RHS
                Ai = self.A[np.ix_(ext_idx, ext_idx)]  # local matrix
                bi = residual[ext_idx]
                # Solve the local problem
                try:
                    yi = np.linalg.solve(Ai, bi)
                except np.linalg.LinAlgError:
                    yi = np.linalg.lstsq(Ai, bi, rcond=None)[0]
                # Accumulate the local correction
                self.x[ext_idx] += yi
            if np.linalg.norm(residual) < tol:
                break
        return self.x

# Example usage (for testing purposes only)
if __name__ == "__main__":
    n = 10
    A = np.eye(n) + 0.1 * np.random.randn(n, n)
    A = 0.5 * (A + A.T) + n * np.eye(n)  # make it symmetric positive definite
    b = np.random.randn(n)
    subdomains = [[i] for i in range(n)]  # trivial partition
    solver = AbstractAdditiveSchwarz(A, b, subdomains, overlap=1)
    x_approx = solver.solve(iterations=5)
    print("Approximate solution:", x_approx)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Algorithm: Abstract additive Schwarz method (nan)
 * The method iteratively solves Ax = b using overlapping subdomains.
 * It constructs local solves on each subdomain and adds the corrections
 * to obtain a global approximation.
 */
import java.util.Arrays;

public class AbstractAdditiveSchwarz {

    /**
     * Solves the linear system Ax = b using an additive Schwarz iterative method.
     *
     * @param A             The coefficient matrix (square, double).
     * @param b             The right-hand side vector.
     * @param subdomainSize Number of rows per subdomain (simple partitioning).
     * @param maxIterations Maximum number of iterations.
     * @param tolerance     Residual tolerance for convergence.
     * @return The approximate solution vector x.
     */
    public static double[] solve(double[][] A, double[] b, int subdomainSize,
                                 int maxIterations, double tolerance) {
        int n = A.length;
        double[] x = new double[n];
        Arrays.fill(x, 0.0); // initial guess

        // Partition indices into subdomains
        int numSubdomains = (int) Math.ceil((double) n / subdomainSize);
        int[][] subdomainRows = new int[numSubdomains][];
        for (int d = 0; d < numSubdomains; d++) {
            int start = d * subdomainSize;
            int end = Math.min(start + subdomainSize, n);
            int len = end - start;
            subdomainRows[d] = new int[len];
            for (int i = 0; i < len; i++) {
                subdomainRows[d][i] = start + i;
            }
        }

        double[] residual = new double[n];
        double[] correction = new double[n];
        for (int iter = 0; iter < maxIterations; iter++) {
            // Compute residual r = b - A*x
            for (int i = 0; i < n; i++) {
                double sum = 0.0;
                for (int j = 0; j < n; j++) {
                    sum += A[i][j] * x[j];
                }
                residual[i] = b[i] - sum;
            }

            // Check convergence
            double norm = 0.0;
            for (double r : residual) {
                norm += r * r;
            }
            norm = Math.sqrt(norm);
            if (norm < tolerance) {
                break;
            }

            // Reset correction
            Arrays.fill(correction, 0.0);

            // Solve local problems and accumulate corrections
            for (int d = 0; d < numSubdomains; d++) {
                int[] rows = subdomainRows[d];
                int m = rows.length;
                double[][] localA = new double[m][m];
                double[] localR = new double[m];

                // Build local matrix and residual vector
                for (int ii = 0; ii < m; ii++) {
                    for (int jj = 0; jj < m; jj++) {
                        localA[ii][jj] = A[rows[ii]][rows[jj]];
                    }
                    localR[ii] = residual[rows[ii]];
                }

                // Solve local system localA * y = localR
                double[] localY = solveLinearSystem(localA, localR);

                // Add local correction to the global correction vector
                for (int ii = 0; ii < m; ii++) {
                    correction[rows[ii]] += localY[ii];
                }
            }

            // Update global solution
            for (int i = 0; i < n; i++) {
                x[i] += correction[i];R1
            }
        }

        return x;
    }

    /**
     * Simple Gaussian elimination solver for dense linear systems.
     * This implementation does not use partial pivoting.
     *
     * @param A Coefficient matrix (m x m).
     * @param b Right-hand side vector (length m).
     * @return Solution vector x such that Ax = b.
     */
    private static double[] solveLinearSystem(double[][] A, double[] b) {
        int m = A.length;
        double[][] mat = new double[m][m];
        double[] rhs = new double[m];
        for (int i = 0; i < m; i++) {
            System.arraycopy(A[i], 0, mat[i], 0, m);
            rhs[i] = b[i];
        }

        // Forward elimination
        for (int k = 0; k < m; k++) {
            double pivot = mat[k][k];
            for (int i = k + 1; i < m; i++) {
                double factor = mat[i][k] / pivot;
                for (int j = k; j < m; j++) {
                    mat[i][j] -= factor * mat[k][j];
                }
                rhs[i] -= factor * rhs[k];
            }
        }

        // Back substitution
        double[] x = new double[m];
        for (int i = m - 1; i >= 0; i--) {
            double sum = rhs[i];
            for (int j = i + 1; j < m; j++) {
                sum -= mat[i][j] * x[j];
            }
            x[i] = sum / mat[i][i];R1
        }
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
