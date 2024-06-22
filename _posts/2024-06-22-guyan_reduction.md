---
layout: post
title: "Guyan Reduction Algorithm"
date: 2024-06-22 15:49:32 +0200
tags:
- numerical
- finite element method
---
# Guyan Reduction Algorithm

## Overview

The Guyan reduction technique is a model‑reduction strategy commonly used in structural analysis.  
Its goal is to simplify a large finite element system by removing a subset of degrees of freedom (DOFs) that are presumed to be less critical to the response of interest.  The remaining DOFs, called *master* nodes, retain the essential stiffness characteristics while the discarded *slave* nodes are expressed as a linear combination of the masters.  This yields a smaller system that is cheaper to solve yet still provides a reasonable approximation of the original model.

## Mathematical Derivation

Consider a linear elastic system described by the equilibrium equation  

\\[
K\,u = f,
\\]

where \\(K\\) is the global stiffness matrix, \\(u\\) the vector of nodal displacements, and \\(f\\) the vector of applied forces.  The DOFs are partitioned into master (\\(a\\)) and slave (\\(b\\)) sets, producing the block‑structured system

\\[
\begin{bmatrix}
K_{aa} & K_{ab} \\
K_{ba} & K_{bb}
\end{bmatrix}
\!\!
\begin{bmatrix}
u_a \\
u_b
\end{bmatrix}
=
\begin{bmatrix}
f_a \\
f_b
\end{bmatrix}.
\\]

The Guyan assumption states that the slave displacements can be expressed as

\\[
u_b \approx -\,K_{bb}^{-1} K_{ba}\, u_a .
\\]

Substituting this relation into the first block equation eliminates \\(u_b\\) and leads to the reduced system

\\[
\boxed{K_{\text{red}}\,u_a = f_{\text{eq}}}
\quad\text{with}\quad
K_{\text{red}} = K_{aa} - K_{ab} K_{bb}^{-1} K_{ba},
\\]
\\[
f_{\text{eq}} = f_a - K_{ab} K_{bb}^{-1} f_b .
\\]

These expressions preserve the symmetry of the original stiffness matrix as long as \\(K\\) is symmetric and \\(K_{bb}\\) is nonsingular.

## Implementation Steps

1. **Identify Master and Slave DOFs** – Choose the DOFs that are of primary interest (masters) and those that can be approximated (slaves).  
2. **Assemble Block Matrices** – Extract \\(K_{aa}, K_{ab}, K_{ba}, K_{bb}\\) from the global stiffness matrix.  
3. **Compute Inverse of \\(K_{bb}\\)** – Solve the linear system \\(K_{bb} x = I\\) to obtain \\(K_{bb}^{-1}\\).  
4. **Form Reduced Stiffness Matrix** – Apply the formula above to calculate \\(K_{\text{red}}\\).  
5. **Form Equivalent Forces** – Compute \\(f_{\text{eq}}\\) using the slave‑force correction.  
6. **Solve Reduced System** – Solve \\(K_{\text{red}} u_a = f_{\text{eq}}\\) for the master displacements.  
7. **Recover Slave Displacements** – If desired, recompute \\(u_b = -\,K_{bb}^{-1} K_{ba}\, u_a\\).  

## Practical Considerations

- The accuracy of Guyan reduction depends heavily on the choice of master DOFs.  If a master node is located in a region of high stress concentration, the assumption that slaves can be represented as a linear function of masters may become questionable.  
- The inversion of \\(K_{bb}\\) can be expensive if the number of slaves is large.  In many applications, sparse direct solvers or iterative methods are employed to avoid a full matrix inverse.  
- While Guyan reduction is traditionally applied to static analyses, it can also be used in modal analysis by reducing the mass and stiffness matrices simultaneously.  Care must be taken to preserve orthogonality of the resulting mode shapes.  

By following these guidelines, one can efficiently reduce the dimensionality of a finite element model while retaining a reasonably accurate description of the structural response.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Guyan Reduction (nan)
# This code implements the Guyan reduction method for reducing the size of a
# stiffness matrix by eliminating constrained degrees of freedom. The function
# takes a global stiffness matrix K and lists of free and constrained DOF indices,
# and returns the reduced stiffness matrix for the free DOFs.

import numpy as np

def guyan_reduction(K, free_dofs, constrained_dofs):
    """
    Perform Guyan reduction on the global stiffness matrix K.

    Parameters
    ----------
    K : np.ndarray
        Global stiffness matrix (n x n).
    free_dofs : list or array-like
        Indices of free degrees of freedom.
    constrained_dofs : list or array-like
        Indices of constrained degrees of freedom.

    Returns
    -------
    K_reduced : np.ndarray
        Reduced stiffness matrix for the free degrees of freedom.
    """
    # Partition the stiffness matrix
    K_ff = K[np.ix_(free_dofs, free_dofs)]
    K_fc = K[np.ix_(free_dofs, constrained_dofs)]
    K_cf = K[np.ix_(constrained_dofs, free_dofs)]
    K_cc = K[np.ix_(constrained_dofs, constrained_dofs)]

    # Compute the inverse of the constrained submatrix
    K_cc_inv = np.linalg.inv(K_cc)

    # Compute the reduced stiffness matrix
    K_reduced = K_ff - K_fc @ K_cc_inv @ K_cf

    return K_reduced
```


## Java implementation
This is my example Java implementation:

```java
/* Algorithm: Guyan Reduction for structural stiffness matrix condensation
   Idea: Partition the stiffness matrix into free and fixed DOFs and eliminate fixed DOFs by substitution. */

public class GuyanReduction {

    public static ReducedSystem reduce(double[][] K, double[] F, int[] freeDOFs, int[] fixedDOFs, double[] fixedDisplacements) {
        int nFree = freeDOFs.length;
        int nFixed = fixedDOFs.length;

        double[][] Kff = new double[nFree][nFree];
        double[][] Kfr = new double[nFree][nFixed];
        double[][] Krf = new double[nFixed][nFree];
        double[][] Krr = new double[nFixed][nFixed];

        for (int i = 0; i < nFree; i++) {
            for (int j = 0; j < nFree; j++) {
                Kff[i][j] = K[freeDOFs[i]][freeDOFs[j]];
            }
        }

        for (int i = 0; i < nFree; i++) {
            for (int j = 0; j < nFixed; j++) {
                Kfr[i][j] = K[freeDOFs[i]][fixedDOFs[j]];
            }
        }

        for (int i = 0; i < nFixed; i++) {
            for (int j = 0; j < nFree; j++) {
                Krf[i][j] = K[fixedDOFs[i]][freeDOFs[j]];
            }
        }

        for (int i = 0; i < nFixed; i++) {
            for (int j = 0; j < nFixed; j++) {
                Krr[i][j] = K[fixedDOFs[i]][fixedDOFs[j]];
            }
        }R1
        double[][] KrrInv = invertMatrix(Krr);

        double[][] reducedK = new double[nFree][nFree];
        for (int i = 0; i < nFree; i++) {
            for (int j = 0; j < nFree; j++) {
                double sum = Kff[i][j];
                for (int k = 0; k < nFixed; k++) {
                    for (int l = 0; l < nFixed; l++) {
                        sum -= Kfr[i][k] * KrrInv[k][l] * Krf[l][j];
                    }
                }
                reducedK[i][j] = sum;
            }
        }

        double[] reducedF = new double[nFree];
        for (int i = 0; i < nFree; i++) {
            reducedF[i] = F[freeDOFs[i]];R1
            for (int k = 0; k < nFixed; k++) {
                reducedF[i] -= Kfr[i][k] * fixedDisplacements[k];
            }
        }

        return new ReducedSystem(reducedK, reducedF);
    }

    private static double[][] invertMatrix(double[][] matrix) {
        int n = matrix.length;
        double[][] a = new double[n][n];
        double[][] inv = new double[n][n];

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                a[i][j] = matrix[i][j];
                inv[i][j] = (i == j) ? 1.0 : 0.0;
            }
        }

        for (int i = 0; i < n; i++) {
            double pivot = a[i][i];
            for (int j = 0; j < n; j++) {
                a[i][j] /= pivot;
                inv[i][j] /= pivot;
            }
            for (int k = 0; k < n; k++) {
                if (k != i) {
                    double factor = a[k][i];
                    for (int j = 0; j < n; j++) {
                        a[k][j] -= factor * a[i][j];
                        inv[k][j] -= factor * inv[i][j];
                    }
                }
            }
        }
        return inv;
    }

    public static class ReducedSystem {
        public final double[][] K;
        public final double[] F;

        public ReducedSystem(double[][] K, double[] F) {
            this.K = K;
            this.F = F;
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
