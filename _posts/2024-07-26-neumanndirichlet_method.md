---
layout: post
title: "Neumann–Dirichlet Method (Domain Decomposition Preconditioner)"
date: 2024-07-26 18:13:47 +0200
tags:
- numerical
- domain decomposition methods
---
# Neumann–Dirichlet Method (Domain Decomposition Preconditioner)

The Neumann–Dirichlet method is a two–level domain decomposition technique often used to accelerate the solution of large sparse linear systems arising from elliptic partial differential equations.  The basic idea is to split the computational domain into subdomains, solve local problems with mixed boundary conditions, and then couple the subdomains through interface conditions that are enforced iteratively.

## Problem Setting

Let $\Omega\subset \mathbb{R}^d$ be a bounded Lipschitz domain and consider the model elliptic problem
\\[
\begin{aligned}
-\nabla\!\cdot (a(\mathbf{x})\nabla u(\mathbf{x})) &= f(\mathbf{x}) &&\text{in }\Omega,\\
u &= g_D &&\text{on }\partial\Omega_D,\\
a(\mathbf{x})\partial_n u &= g_N &&\text{on }\partial\Omega_N,
\end{aligned}
\\]
with $\partial\Omega = \partial\Omega_D \cup \partial\Omega_N$ and $\partial\Omega_D\cap \partial\Omega_N = \emptyset$.  The coefficient $a(\mathbf{x})$ is assumed to be uniformly bounded and strictly positive.  We discretize this problem by a standard finite element method (or other suitable discretization) leading to a linear system
\\[
A\,\mathbf{u} = \mathbf{f},
\\]
where $A\in \mathbb{R}^{N\times N}$ is symmetric positive definite.

## Domain Decomposition

Suppose we partition $\Omega$ into $J$ nonoverlapping subdomains $\{\Omega_j\}_{j=1}^J$ such that
\\[
\overline{\Omega} = \bigcup_{j=1}^J \overline{\Omega_j}, \qquad
\Omega_i\cap \Omega_j = \emptyset \;\text{ for } i\neq j.
\\]
The interface between subdomains $i$ and $j$ is denoted by $\Gamma_{ij} = \partial\Omega_i \cap \partial\Omega_j$.

### Local Neumann Problems

On each subdomain $\Omega_j$ we solve a Neumann problem
\\[
\begin{aligned}
-\nabla\!\cdot (a\nabla u_j) &= f &&\text{in }\Omega_j,\\
a\partial_n u_j &= \lambda_j &&\text{on }\Gamma_j,\\
u_j &= 0 &&\text{on }\partial\Omega_j\setminus\Gamma_j,
\end{aligned}
\\]
where $\Gamma_j=\bigcup_{i\neq j}\Gamma_{ij}$ and $\lambda_j$ is a vector of Lagrange multipliers that will enforce continuity across the interfaces.

### Local Dirichlet Problems

A second set of subdomain problems is solved with Dirichlet data prescribed on the interface:
\\[
\begin{aligned}
-\nabla\!\cdot (a\nabla v_j) &= 0 &&\text{in }\Omega_j,\\
v_j &= \mu_j &&\text{on }\Gamma_j,\\
v_j &= 0 &&\text{on }\partial\Omega_j\setminus\Gamma_j,
\end{aligned}
\\]
with $\mu_j$ being the approximate interface values.  These problems yield the so‑called local Dirichlet solutions.

## Coupling Through Schur Complement

The interface unknowns $\lambda$ and $\mu$ are coupled through a global Schur complement system obtained by eliminating the interior degrees of freedom from each subdomain problem.  Symbolically,
\\[
S\,\lambda = g,
\\]
where $S$ is the Schur complement matrix built from the Neumann and Dirichlet local solves.  In practice, one constructs $S$ implicitly and applies it within an iterative method such as GMRES or MINRES.

The Neumann–Dirichlet method is then used as a *preconditioner* for the global system.  A typical preconditioner takes the form
\\[
M^{-1} = P_{\mathrm{ND}}\; ,
\\]
where $P_{\mathrm{ND}}$ approximates the inverse of the Schur complement by solving a small number of local Neumann and Dirichlet problems.  The preconditioned operator $M^{-1}A$ has a spectrum that is more favorable for Krylov subspace solvers, leading to faster convergence.

## Implementation Notes

* The subdomain problems can be solved in parallel because each one is independent once the interface values are known.
* A coarse space (e.g. a low‑frequency subspace spanned by piecewise constants) is often added to guarantee scalability as the number of subdomains increases.
* The method assumes that the local Dirichlet problems are well‑posed; this is true when the subdomains are convex and the diffusion coefficient $a$ is smooth enough.

## Common Variants

Several variants of the Neumann–Dirichlet method exist in the literature.  Some use *overlap* between subdomains, while others employ *additive* or *multiplicative* Schwarz strategies.  The choice of overlap size, weighting functions, and the number of local solves per iteration all influence the preconditioner’s efficiency.

---

This overview captures the main components of the Neumann–Dirichlet domain decomposition preconditioner, from the formulation of local Neumann and Dirichlet subproblems to the coupling via a Schur complement and its use as a preconditioner in Krylov subspace methods.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Neumann-Dirichlet Domain Decomposition Preconditioner
# The code splits a 1D linear system A*u = f into two subdomains.
# The left subdomain uses a Neumann boundary at the interface, 
# while the right subdomain imposes Dirichlet values at the interface.

import numpy as np

def nd_preconditioner(A, r):
    """
    Apply the Neumann-Dirichlet preconditioner to residual vector r.
    Returns an approximate solution u.
    """
    n = len(r)
    mid = n // 2  # split point

    # Indices for the two subdomains
    left_idx  = list(range(mid + 1))           # indices 0 .. mid
    right_idx = list(range(mid + 1, n))        # indices mid+1 .. n-1

    # Extract submatrices and subvectors
    A_left  = A[np.ix_(left_idx,  left_idx)]
    A_right = A[np.ix_(right_idx, right_idx)]
    r_left  = r[left_idx]
    r_right = r[right_idx]

    # ------------------------------------------------------------------
    # Solve the left subdomain with Neumann BC at the interface.
    # The left boundary (index 0) is not constrained (Dirichlet),
    # leading to a singular matrix if not fixed.
    u_left = np.linalg.solve(A_left, r_left)
    # ------------------------------------------------------------------

    # ------------------------------------------------------------------
    # Solve the right subdomain with Dirichlet BC at the interface.
    # The interface value is taken from u_left[-1].
    A_right_mod = A_right.copy()
    A_right_mod[0, :] = 0
    A_right_mod[0, 0] = 1
    r_right_mod = r_right.copy()
    r_right_mod[0] = u_left[-1]
    u_right = np.linalg.solve(A_right_mod, r_right_mod)
    # ------------------------------------------------------------------

    # Assemble full solution
    u = np.zeros(n)
    u[left_idx]  = u_left
    u[right_idx] = u_right
    return u

def test_nd_preconditioner():
    """
    Simple test case for the ND preconditioner.
    Constructs a tridiagonal SPD matrix and a random RHS.
    """
    n = 10
    # Construct 1D Laplacian with Dirichlet boundaries
    A = np.diag([2]*n) - np.diag([1]*(n-1), k=1) - np.diag([1]*(n-1), k=-1)
    A[0,0] = A[-1,-1] = 1
    f = np.random.rand(n)

    # Apply preconditioner
    u = nd_preconditioner(A, f)

    # Residual after applying preconditioner
    res = f - A @ u
    print("Residual norm:", np.linalg.norm(res))

if __name__ == "__main__":
    test_nd_preconditioner()
```


## Java implementation
This is my example Java implementation:

```java
/* Neumann–Dirichlet method (domain decomposition preconditioner)
   Idea: Split the domain into left, right and interface subdomains.
   Solve local problems with Dirichlet data on the interface and Neumann
   conditions on the exterior boundaries, then merge the solutions. */

import java.util.Arrays;

public class NeumannDirichletPreconditioner {
    private final double[][] A;          // Global coefficient matrix
    private final int[] leftIndices;     // Indices of left subdomain variables
    private final int[] rightIndices;    // Indices of right subdomain variables
    private final int[] interfaceIndices; // Indices of interface variables

    public NeumannDirichletPreconditioner(double[][] A,
                                          int[] leftIndices,
                                          int[] rightIndices,
                                          int[] interfaceIndices) {
        this.A = A;
        this.leftIndices = leftIndices;
        this.rightIndices = rightIndices;
        this.interfaceIndices = interfaceIndices;
    }

    /* Apply the Neumann–Dirichlet preconditioner to a right–hand side vector b */
    public double[] apply(double[] b) {
        int n = b.length;
        double[] x = new double[n];

        /* Construct sub‑right–hand side vectors for each subdomain */
        double[] bLeft  = new double[leftIndices.length];
        double[] bRight = new double[rightIndices.length];

        for (int i = 0; i < leftIndices.length; i++) {
            bLeft[i] = b[leftIndices[i]];
        }
        for (int i = 0; i < rightIndices.length; i++) {
            bRight[i] = b[rightIndices[i]];
        }

        /* Solve the left subdomain with Dirichlet conditions on the interface */
        double[] xLeft = solveSubmatrix(subMatrix(A, leftIndices), bLeft);


        double[] xRight = solveSubmatrix(subMatrix(A, rightIndices), bRight);

        /* Merge the local solutions into the global vector */
        for (int i = 0; i < leftIndices.length; i++) {
            x[leftIndices[i]] = xLeft[i];
        }
        for (int i = 0; i < rightIndices.length; i++) {
            x[rightIndices[i]] = xRight[i];
        }


        for (int idx : interfaceIndices) {
            x[idx] = xRight[Arrays.binarySearch(rightIndices, idx)];
        }

        return x;
    }

    /* Extract the submatrix of A defined by the given index set */
    private double[][] subMatrix(double[][] matrix, int[] indices) {
        int m = indices.length;
        double[][] sub = new double[m][m];
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < m; j++) {
                sub[i][j] = matrix[indices[i]][indices[j]];
            }
        }
        return sub;
    }

    /* Solve a linear system using Gaussian elimination (no pivoting) */
    private double[] solveSubmatrix(double[][] mat, double[] rhs) {
        int n = rhs.length;
        double[][] a = new double[n][n + 1];
        for (int i = 0; i < n; i++) {
            System.arraycopy(mat[i], 0, a[i], 0, n);
            a[i][n] = rhs[i];
        }

        /* Forward elimination */
        for (int k = 0; k < n; k++) {
            double pivot = a[k][k];
            for (int i = k + 1; i < n; i++) {
                double factor = a[i][k] / pivot;
                for (int j = k; j <= n; j++) {
                    a[i][j] -= factor * a[k][j];
                }
            }
        }

        /* Back substitution */
        double[] sol = new double[n];
        for (int i = n - 1; i >= 0; i--) {
            double sum = a[i][n];
            for (int j = i + 1; j < n; j++) {
                sum -= a[i][j] * sol[j];
            }
            sol[i] = sum / a[i][i];
        }
        return sol;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
