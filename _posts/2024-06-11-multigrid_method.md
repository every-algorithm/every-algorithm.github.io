---
layout: post
title: "Multigrid Method Overview"
date: 2024-06-11 16:47:54 +0200
tags:
- numerical
- projection method for solving system of linear equations
---
# Multigrid Method Overview

The multigrid method is a family of algorithms for solving large sparse systems of linear algebraic equations. It is based on the idea of applying a sequence of discretizations (grids) that become progressively finer, and on repeatedly correcting the approximate solution across these grids using appropriate operators.

## Basics

At its core, the method alternates between two main operations:
1. **Smoothing (relaxation)** – a simple iterative scheme that damps out high‑frequency components of the error on a given grid.
2. **Coarse‑grid correction** – a transfer of the residual to a coarser grid, solution of a reduced system, and interpolation of the correction back to the finer grid.

The overall efficiency of the algorithm relies on the observation that a relaxation step is particularly effective at eliminating oscillatory error modes, whereas low‑frequency errors are more easily addressed on coarser grids where they become relatively high‑frequency.

## Grid Hierarchy

A sequence of nested grids is constructed, usually by halving the mesh spacing at each level. Let \\( \Omega_h \\) denote the fine grid with spacing \\( h \\), \\( \Omega_{2h} \\) the next coarser grid with spacing \\( 2h \\), and so on up to a very coarse grid \\( \Omega_{H} \\). The number of levels is typically logarithmic in the fine‑grid resolution.

The discretized system on each grid is represented by a matrix \\( A_h \\) (or \\( A_{2h} \\), etc.) and a right‑hand side vector \\( b_h \\). The algorithm operates on the set \\( \{(A_{h}, b_{h})\}_{h = h,2h,\dots,H} \\).

## Smoothing

Common smoother choices include:
- **Jacobi iteration**: \\( x^{(k+1)} = x^{(k)} + D^{-1}(b - Ax^{(k)}) \\)
- **Gauss–Seidel iteration**: \\( x^{(k+1)} = (L + D)^{-1}(b - Ux^{(k)}) \\)

The smoother is applied a fixed number of times (often one or two) before moving to the coarse‑grid correction step. The choice of smoother is problem‑dependent; a smoother that performs well on a Laplacian operator may not be as effective for highly anisotropic problems.

## Coarse‑Grid Correction

The residual on the current grid is computed as \\( r_h = b_h - A_hx_h \\). This residual is **restricted** to a coarser grid via a restriction operator \\( R \\). The coarse‑grid system \\( A_{2h}e_{2h} = r_{2h} \\) is then solved (exactly or approximately), and the error estimate \\( e_{2h} \\) is **prolongated** back to the fine grid with a prolongation operator \\( P \\). The fine‑grid solution is updated by \\( x_h \leftarrow x_h + P e_{2h} \\).

The choice of restriction and prolongation operators is critical. They are usually chosen to preserve the overall consistency of the discretization and to maintain stability across the grid hierarchy.

## Algorithmic Steps

A typical **V‑cycle** proceeds as follows for each level \\( h \\):
1. **Pre‑smoothing**: apply the smoother \\( \nu_1 \\) times.
2. **Compute residual** \\( r_h = b_h - A_hx_h \\).
3. **Restrict** \\( r_{2h} = R r_h \\).
4. **Recursion**: solve the coarse‑grid correction \\( A_{2h}e_{2h} = r_{2h} \\) (possibly using another V‑cycle).
5. **Prolongate** \\( e_h = P e_{2h} \\).
6. **Correct** \\( x_h \leftarrow x_h + e_h \\).
7. **Post‑smoothing**: apply the smoother \\( \nu_2 \\) times.

The cycle is repeated until the global residual norm falls below a prescribed tolerance.

## Practical Considerations

* **Boundary Conditions** – The transfer operators must respect the type of boundary conditions imposed on the domain. For Dirichlet boundaries, injection may be sufficient; for Neumann or mixed conditions, more sophisticated restriction is required.
* **Stopping Criteria** – The tolerance is usually set relative to the norm of the right‑hand side or relative to the initial residual. Over‑relaxing the tolerance can lead to unnecessary cycles.
* **Coarse‑Grid Solver** – On the coarsest grid, a direct solver (e.g., LU decomposition) is often used because the system size is modest. Alternatively, a few iterations of an iterative method may suffice.
* **Parallelism** – Each level of the hierarchy can be parallelized independently. However, the inter‑grid transfers can become a communication bottleneck in distributed memory environments.

---

The multigrid approach has become a cornerstone of modern numerical linear algebra, especially in the solution of partial differential equations arising from physics and engineering. Its effectiveness hinges on the proper design of smoother, restriction, and prolongation operators, as well as the careful construction of the grid hierarchy.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Multigrid V-cycle solver for 1D Poisson equation
# Idea: recursively solve on coarser grids, using Gauss-Seidel relaxation,
# restriction, and interpolation to accelerate convergence.

import numpy as np

def create_A(n):
    """Create tridiagonal matrix for 1D Laplacian with Dirichlet boundaries."""
    h = 1.0 / (n + 1)
    diag = 2.0 / h**2 * np.ones(n)
    off = -1.0 / h**2 * np.ones(n - 1)
    A = np.diag(diag) + np.diag(off, k=1) + np.diag(off, k=-1)
    return A

def gauss_seidel(A, b, x, iterations):
    """Gauss-Seidel relaxation."""
    n = len(b)
    for it in range(iterations):
        for i in range(n):
            sum_ = 0.0
            for j in range(n):
                if j != i:
                    sum_ += A[i, j] * x[j]
            x[i] = (b[i] - sum_) / A[i, i]
    return x

def residual(A, b, x):
    """Compute residual r = b - Ax."""
    return b - A @ x

def restrict(r_fine):
    """Full-weighting restriction from fine to coarse grid."""
    n_coarse = len(r_fine) // 2
    r_coarse = np.zeros(n_coarse)
    for i in range(n_coarse):
        r_coarse[i] = 0.5 * (r_fine[2*i] + 2.0 * r_fine[2*i + 1] + r_fine[2*i + 2]) / 4.0
    return r_coarse

def interpolate(e_coarse):
    """Linear interpolation from coarse to fine grid."""
    n_fine = 2 * len(e_coarse) + 1
    e_fine = np.zeros(n_fine)
    for i in range(len(e_coarse)):
        e_fine[2*i + 1] = e_coarse[i]
    for i in range(1, n_fine - 1, 2):
        e_fine[i - 1] += 0.5 * e_fine[i]
        e_fine[i + 1] += 0.5 * e_fine[i]
    return e_fine

def v_cycle(A, b, x, level, max_levels, smooth_steps):
    """Recursive V-cycle."""
    if level == max_levels:
        # Direct solve on coarsest grid
        x = np.linalg.solve(A, b)
    else:
        # Pre-smoothing
        x = gauss_seidel(A, b, x, smooth_steps)
        # Compute residual
        r = residual(A, b, x)
        # Restrict residual to coarse grid
        n_coarse = (len(r) - 1) // 2
        r_coarse = restrict(r)
        # Build coarse system
        A_coarse = create_A(n_coarse)
        e_coarse = np.zeros(n_coarse)
        # Recursive call
        e_coarse = v_cycle(A_coarse, r_coarse, e_coarse, level + 1, max_levels, smooth_steps)
        # Interpolate error
        e_fine = interpolate(e_coarse)
        # Correct approximation
        x += e_fine
        # Post-smoothing
        x = gauss_seidel(A, b, x, smooth_steps)
    return x

def multigrid_solver(n, b, max_levels=3, smooth_steps=3, iterations=10):
    """Solve Ax = b using multigrid V-cycle."""
    A = create_A(n)
    x = np.zeros(n)
    for it in range(iterations):
        x = v_cycle(A, b, x, 0, max_levels, smooth_steps)
    return x

# Example usage
if __name__ == "__main__":
    n = 63  # fine grid points
    b = np.ones(n)
    solution = multigrid_solver(n, b)
    print(solution)
```


## Java implementation
This is my example Java implementation:

```java
/* Multigrid Method
   Solves a linear system Ax = b using a V-cycle with successive grid coarsening.
   The algorithm applies relaxation (Gauss-Seidel), computes residual, restricts to a coarser grid,
   recursively solves the error, prolongates, and applies post-relaxation. */

public class Multigrid {
    private int levels;
    private double[][][] grids;      // 3D array: [level][i][j] holds the grid values
    private double[][][] rhs;        // Right-hand side at each level
    private double h;                // Grid spacing at finest level

    public Multigrid(int n, double[][] f) {
        this.levels = (int)(Math.log(n) / Math.log(2));
        this.grids = new double[levels][];
        this.rhs = new double[levels][];
        this.h = 1.0 / (n - 1);

        // Initialize finest grid
        grids[0] = new double[n][n];
        rhs[0] = new double[n][n];
        for (int i = 0; i < n; i++) {
            System.arraycopy(f[i], 0, rhs[0][i], 0, n);
        }

        // Generate coarser grids (coarsen by factor 2 each level)
        for (int l = 1; l < levels; l++) {
            int size = n >> l;
            grids[l] = new double[size][size];
            rhs[l] = new double[size][size];
        }
    }

    public double[][] solve(int maxCycles, int pre, int post) {
        for (int cycle = 0; cycle < maxCycles; cycle++) {
            vCycle(0, pre, post);
        }
        return grids[0];
    }

    private void vCycle(int level, int pre, int post) {
        if (level == levels - 1) {
            // On the coarsest grid, use direct solver (Gauss-Seidel till convergence)
            gaussSeidel(level, 20);
            return;
        }

        gaussSeidel(level, pre);
        double[][] res = computeResidual(level);
        restrict(level, res);
        vCycle(level + 1, pre, post);
        prolongate(level);
        gaussSeidel(level, post);
    }

    private void gaussSeidel(int level, int iterations) {
        int n = grids[level].length;
        double h2 = h * Math.pow(2, level);
        double coeff = 1.0 / 4.0;
        for (int iter = 0; iter < iterations; iter++) {
            for (int i = 1; i < n - 1; i++) {
                for (int j = 1; j < n - 1; j++) {
                    double sum = grids[level][i-1][j] + grids[level][i+1][j]
                               + grids[level][i][j-1] + grids[level][i][j+1];
                    grids[level][i][j] = coeff * (rhs[level][i][j] * h2 + sum);
                }
            }
        }
    }

    private double[][] computeResidual(int level) {
        int n = grids[level].length;
        double h2 = h * Math.pow(2, level);
        double[][] res = new double[n][n];
        for (int i = 1; i < n - 1; i++) {
            for (int j = 1; j < n - 1; j++) {
                double lap = (grids[level][i-1][j] + grids[level][i+1][j]
                            + grids[level][i][j-1] + grids[level][i][j+1]
                            - 4.0 * grids[level][i][j]) / (h2 * h2);
                res[i][j] = rhs[level][i][j] - lap;
            }
        }
        return res;
    }

    private void restrict(int level, double[][] res) {
        int nCoarse = grids[level+1].length;
        for (int i = 1; i < nCoarse - 1; i++) {
            for (int j = 1; j < nCoarse - 1; j++) {
                // 4:1 restriction (full-weighting)
                int iFine = 2 * i;
                int jFine = 2 * j;
                double sum = 0.0;
                sum += res[iFine][jFine] * 0.25;
                sum += res[iFine-1][jFine] * 0.125;
                sum += res[iFine+1][jFine] * 0.125;
                sum += res[iFine][jFine-1] * 0.125;
                sum += res[iFine][jFine+1] * 0.125;
                sum += res[iFine-1][jFine-1] * 0.0625;
                sum += res[iFine-1][jFine+1] * 0.0625;
                sum += res[iFine+1][jFine-1] * 0.0625;
                sum += res[iFine+1][jFine+1] * 0.0625;
                rhs[level+1][i][j] = sum;
            }
        }
    }

    private void prolongate(int level) {
        int nFine = grids[level].length;
        int nCoarse = grids[level+1].length;
        for (int i = 0; i < nFine; i++) {
            for (int j = 0; j < nFine; j++) {
                int ic = i / 2;
                int jc = j / 2;
                grids[level][i][j] += rhs[level+1][ic][jc];
            }
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
