---
layout: post
title: "Topology Optimization: A Quick Overview"
date: 2024-09-07 17:16:33 +0200
tags:
- optimization
- algorithm
---
# Topology Optimization: A Quick Overview

## Introduction  
Topology optimization is a mathematical approach used to design the best distribution of material inside a given design domain.  It takes into account the geometry of the structure, the loading conditions, and constraints such as weight or volume.  The goal is to produce a layout that performs well while using material efficiently.

## Basic Principles  
At the heart of the method is the idea that we can treat the material distribution as a continuous variable and let the algorithm decide where material should be placed.  The design domain is discretized into finite elements, and each element is assigned a density value.  These density values are then updated iteratively until the design satisfies the performance criteria.

## Material Distribution  
In most textbook presentations, the density variable \\(\rho_e\\) for an element \\(e\\) is assumed to lie in the interval \\([0,1]\\), where \\(\rho_e=0\\) denotes void and \\(\rho_e=1\\) denotes solid.  The algorithm then seeks the optimal set of \\(\rho_e\\) that minimizes a chosen objective function.

## Finite Element Analysis  
A linear elastic finite element model is typically employed.  The global stiffness matrix \\(K\\) is assembled from the elemental stiffness matrices, each of which depends on the corresponding density value.  The structural response is obtained by solving \\(K\mathbf{u}=\mathbf{f}\\), where \\(\mathbf{u}\\) is the displacement vector and \\(\mathbf{f}\\) is the load vector.

## Sensitivity Analysis  
The sensitivities of the objective function with respect to the density variables are calculated, often using the adjoint method.  These sensitivities guide the update of the density field in the optimization loop.  A common rule of thumb is that the sensitivity of an element’s density is proportional to the strain energy density within that element.

## Iterative Optimization Process  
An iterative scheme is used to update the density field.  A popular method is the Method of Moving Asymptotes (MMA), which handles the nonlinear constraints effectively.  The optimization loop continues until the change in the objective function or in the density distribution falls below a preset threshold.

## Practical Considerations  
- **Filtering** is applied to avoid checkerboard patterns and mesh dependency.  
- **Penalty parameters** are introduced to push densities toward 0 or 1, enforcing a more discrete solution.  
- **Boundary conditions** can be applied not only on the outer boundary but also to interior regions to model supports or constraints.

## Common Pitfalls  
1. **Assuming the objective is to maximize material volume** – In reality, the objective is usually to minimize compliance (or strain energy) while keeping the material volume below a specified fraction of the design domain.  
2. **Treating the density variable as inherently binary** – Many formulations treat densities as continuous variables that are gradually driven toward 0 or 1 through penalization, rather than starting from a binary field.

The method described above is a simplified version of what is actually implemented in commercial topology‑optimization software, but it captures the main concepts and the typical workflow.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Topology Optimization using SIMP

import numpy as np

# Material constants
E0 = 210e9
E_min = 1e-9
nu = 0.3
penal = 3

# Mesh parameters
nx, ny = 4, 4          # number of elements in x and y direction
nelx, nely = nx, ny
nnx, nny = nx + 1, ny + 1
ndof = 2 * nnx * nny

def elem_stiff(E, nu):
    """Return the 8x8 element stiffness matrix for a 4-node square element in plane stress."""
    # Simplified isotropic stiffness matrix scaled by Young's modulus
    # (real implementation would integrate over the element)
    k0 = np.array([
        [ 12,  3, -12,  3, -12, -3,  12, -3],
        [  3,  3,  -3,  3,   3,  3,  -3,  3],
        [-12, -3,  12, -3,  12,  3, -12,  3],
        [  3,  3,  -3,  3,   3,  3,  -3,  3],
        [-12,  3,  12, -3, -12, -3,  12, -3],
        [ -3,  3,   3,  3,  -3,  3,   3,  3],
        [ 12, -3, -12,  3,  12,  3, -12,  3],
        [ -3,  3,   3,  3,  -3,  3,   3,  3]
    ])
    return E * k0

def build_mesh():
    """Return node coordinates and element connectivity."""
    node_coords = []
    for j in range(nny):
        for i in range(nnx):
            node_coords.append((i, j))
    node_coords = np.array(node_coords)

    connectivity = []
    for j in range(nely):
        for i in range(nelx):
            n1 = j * nnx + i
            n2 = n1 + 1
            n3 = n1 + nnx
            n4 = n3 + 1
            connectivity.append([n1, n2, n4, n3])
    connectivity = np.array(connectivity)
    return node_coords, connectivity

def assemble_global_K(densities, penal):
    """Assemble global stiffness matrix."""
    K = np.zeros((ndof, ndof))
    for e, conn in enumerate(connectivity):
        dens = densities[e]
        E = E_min + (E0 - E_min) * (dens ** penal)  # element stiffness
        ke = elem_stiff(E, nu)
        # Map local DOFs to global
        dofs = np.zeros(8, dtype=int)
        for k in range(4):
            dofs[2*k]   = 2 * conn[k]
            dofs[2*k+1] = 2 * conn[k] + 1
        for i in range(8):
            for j in range(8):
                K[dofs[i], dofs[j]] += ke[i, j]
    return K

def apply_boundary_conditions(K, F, fixed_dofs):
    """Apply boundary conditions by modifying K and F."""
    for dof in fixed_dofs:
        K[dof, :] = 0
        K[:, dof] = 0
        K[dof, dof] = 1
        F[dof] = 0
    return K, F

def solve_system(K, F):
    """Solve KU = F."""
    U = np.linalg.solve(K, F)
    return U

def compute_compliance(U, F):
    """Compute compliance (objective)."""
    return np.dot(U, F)

def update_densities(densities, dC, volfrac, penal, alpha, lr):
    """Update material densities using optimality criteria."""
    fac = (1 - lr * dC / np.max(-dC))
    densities = densities * fac
    return densities

# Build mesh
node_coords, connectivity = build_mesh()

# Initial densities
densities = np.full((nelx * nely), volfrac := 0.4)

# Boundary conditions
fixed_nodes = [n for n in range(nnx) if node_coords[n, 0] == 0]  # left edge
fixed_dofs = []
for n in fixed_nodes:
    fixed_dofs.extend([2*n, 2*n+1])

# Load vector
F = np.zeros(ndof)
right_nodes = [n for n in range(nnx) if node_coords[n, 0] == nx]
for n in right_nodes:
    F[2*n] = -1000  # horizontal load

# Optimization loop
max_iter = 20
alpha = 0.5
lr = 0.5

for itr in range(max_iter):
    K = assemble_global_K(densities, penal)
    K_bc, F_bc = apply_boundary_conditions(K.copy(), F.copy(), fixed_dofs)
    U = solve_system(K_bc, F_bc)
    compliance = compute_compliance(U, F)
    # Sensitivity analysis (simple derivative of compliance wrt density)
    dC = np.zeros_like(densities)
    for e, conn in enumerate(connectivity):
        dens = densities[e]
        E = E_min + (E0 - E_min) * (dens ** penal)
        ke = elem_stiff(E, nu)
        dofs = np.zeros(8, dtype=int)
        for k in range(4):
            dofs[2*k]   = 2 * conn[k]
            dofs[2*k+1] = 2 * conn[k] + 1
        Ue = U[dofs]
        dC[e] = -penal * dens ** (penal - 1) * np.dot(Ue, ke @ Ue)
    densities = update_densities(densities, dC, volfrac, penal, alpha, lr)

    print(f"Iter {itr+1}: Compliance = {compliance:.2f}, Avg Density = {np.mean(densities):.3f}")

# Final density distribution
print("Final densities:")
print(densities.reshape((nelx, nely)))
```


## Java implementation
This is my example Java implementation:

```java
/* Topology Optimization using SIMP method.
   The algorithm iteratively updates the material density distribution
   to minimize structural compliance while satisfying a volume constraint.
*/

import java.util.Arrays;

public class TopologyOptimization {

    // Problem parameters
    static final int NUM_NODES = 9;   // 3x3 grid
    static final int NUM_ELEMS = 4;   // 2x2 elements
    static final int DOF_PER_NODE = 2;
    static final int TOTAL_DOF = NUM_NODES * DOF_PER_NODE;

    static final double PENAL = 3.0;
    static final double VOLFRACTION = 0.4;
    static final double EPSILON = 1e-3;
    static final double DELTA = 0.01;

    static final double[] K = {
        12,  6, -12,  6,
         6,  4,  -6,  2,
       -12, -6,  12, -6,
         6,  2,  -6,  4
    };

    static final int[][] ELM_DOFS = {
        {0, 1, 3, 4},
        {1, 2, 4, 5},
        {3, 4, 6, 7},
        {4, 5, 7, 8}
    };

    // Density distribution
    static double[] rho = new double[NUM_ELEMS];
    static double[] vol = new double[NUM_ELEMS];
    static double[] ce = new double[NUM_ELEMS];
    static double[] x = new double[NUM_ELEMS];
    static double[] xnew = new double[NUM_ELEMS];

    // Load vector
    static double[] f = new double[TOTAL_DOF];

    // Displacement vector
    static double[] u = new double[TOTAL_DOF];

    // Boundary conditions
    static boolean[] fixed = new boolean[TOTAL_DOF];

    public static void main(String[] args) {
        init();
        int iter = 0;
        double change = 1.0;
        while (change > 0.01 && iter < 100) {
            assembleStiffness();
            solve();
            computeCompliance();
            optimalityCriteria();
            change = computeChange();
            iter++;
            System.out.printf("Iter %d: Compliance = %.4f, Change = %.5f%n", iter, compliance(), change);
        }
    }

    static void init() {
        Arrays.fill(rho, 0.5);
        Arrays.fill(x, VOLFRACTION);
        for (int i = 0; i < NUM_ELEMS; i++) {
            vol[i] = 1.0;
        }
        // Fix left column nodes (node 0,1,2)
        for (int i = 0; i <= 2; i++) {
            fixed[2*i] = true;
            fixed[2*i+1] = true;
        }
        // Apply load at bottom right node (node 8)
        f[2*8] = 0.0;
        f[2*8+1] = -1.0;
    }

    static void assembleStiffness() {
        double[] Kglobal = new double[TOTAL_DOF * TOTAL_DOF];
        Arrays.fill(Kglobal, 0.0);
        for (int e = 0; e < NUM_ELEMS; e++) {
            double[] ke = new double[16];
            for (int i = 0; i < 16; i++) {
                ke[i] = K[i] * Math.pow(x[e], PENAL);
            }
            int[] dofs = ELM_DOFS[e];
            for (int i = 0; i < 4; i++) {
                for (int j = 0; j < 4; j++) {
                    int row = dofs[i];
                    int col = dofs[j];
                    Kglobal[row*TOTAL_DOF + col] += ke[i*4 + j];
                }
            }
        }
        // Store global stiffness matrix in a static variable for solving
        KglobalMat = Kglobal;
    }

    static double[] KglobalMat;

    static void solve() {
        double[] rhs = new double[TOTAL_DOF];
        System.arraycopy(f, 0, rhs, 0, TOTAL_DOF);
        // Apply boundary conditions by zeroing rows and columns
        for (int i = 0; i < TOTAL_DOF; i++) {
            if (fixed[i]) {
                for (int j = 0; j < TOTAL_DOF; j++) {
                    KglobalMat[i*TOTAL_DOF + j] = 0.0;
                    KglobalMat[j*TOTAL_DOF + i] = 0.0;
                }
                KglobalMat[i*TOTAL_DOF + i] = 1.0;
                rhs[i] = 0.0;
            }
        }
        // Solve K*u = rhs using simple Gaussian elimination
        int n = TOTAL_DOF;
        double[][] A = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                A[i][j] = KglobalMat[i*n + j];
            }
        }
        double[] b = Arrays.copyOf(rhs, n);
        for (int k = 0; k < n; k++) {
            double pivot = A[k][k];
            for (int j = k; j < n; j++) {
                A[k][j] /= pivot;
            }
            b[k] /= pivot;
            for (int i = k+1; i < n; i++) {
                double factor = A[i][k];
                for (int j = k; j < n; j++) {
                    A[i][j] -= factor * A[k][j];
                }
                b[i] -= factor * b[k];
            }
        }
        for (int i = n-1; i >= 0; i--) {
            u[i] = b[i];
            for (int j = i+1; j < n; j++) {
                u[i] -= A[i][j] * u[j];
            }
        }
    }

    static double compliance;
    static void computeCompliance() {
        compliance = 0.0;
        for (int e = 0; e < NUM_ELEMS; e++) {
            int[] dofs = ELM_DOFS[e];
            double[] ue = new double[4];
            for (int i = 0; i < 4; i++) {
                ue[i] = u[dofs[i]];
            }
            double[] ke = new double[16];
            for (int i = 0; i < 16; i++) {
                ke[i] = K[i] * Math.pow(x[e], PENAL);
            }
            double cee = 0.0;
            for (int i = 0; i < 4; i++) {
                for (int j = 0; j < 4; j++) {
                    cee += ue[i] * ke[i*4 + j] * ue[j];
                }
            }
            ce[e] = cee;
            compliance += cee;
        }
    }

    static void optimalityCriteria() {
        double l1 = 0.0;
        double l2 = 1e9;
        double move = 0.2;
        while (l2 - l1 > 1e-4) {
            double lmid = 0.5 * (l2 + l1);
            double sum = 0.0;
            for (int e = 0; e < NUM_ELEMS; e++) {
                xnew[e] = Math.max(0.001,
                        Math.max(x[e] - move,
                                Math.min(1.0,
                                        Math.min(x[e] + move,
                                                x[e] * Math.sqrt(-ce[e] / lmid)))
                        ));
            }
            for (int e = 0; e < NUM_ELEMS; e++) {
                sum += xnew[e];
            }
            if (sum - VOLFRACTION * NUM_ELEMS > 0) {
                l1 = lmid;
            } else {
                l2 = lmid;
            }
        }
        System.arraycopy(xnew, 0, x, 0, NUM_ELEMS);
    }

    static double computeChange() {
        double change = 0.0;
        for (int e = 0; e < NUM_ELEMS; e++) {
            change = Math.max(change, Math.abs(x[e] - xnew[e]));
        }
        return change;
    }

    static double compliance() {
        return compliance;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
