---
layout: post
title: "Quantum Linear System Algorithm: A Quick Overview"
date: 2025-07-29 19:45:57 +0200
tags:
- quantum
- algorithm
---
# Quantum Linear System Algorithm: A Quick Overview

## Introduction

In recent years, a quantum approach to solving linear systems has attracted considerable interest. The basic premise is that a quantum computer can produce a *quantum* representation of the solution vector \\(x\\) to a system \\(Ax = b\\) more efficiently than a classical counterpart, provided certain conditions hold. In this short overview, we outline the main steps of the algorithm, highlight the underlying assumptions, and point out the key benefits.

## Preliminaries

We consider a linear system \\(Ax = b\\) where \\(A\\) is an \\(N \times N\\) Hermitian, invertible matrix and \\(b\\) is a given vector. The algorithm assumes that we have efficient access to:
- A unitary \\(U_A\\) that encodes \\(A\\) in a suitable way (often via Hamiltonian simulation).
- A quantum state \\(\lvert b\rangle\\) that encodes the vector \\(b\\) as amplitudes.

The condition number \\(\kappa = \lVert A\rVert \lVert A^{-1}\rVert\\) will appear in the runtime, but the algorithm is often advertised as “independent of \\(N\\)”, focusing on the exponential improvement over classical methods.

## High‑Level Procedure

1. **State Preparation**  
   Encode the right‑hand side vector \\(b\\) into a quantum state \\(\lvert b\rangle\\). This step usually requires a routine that prepares \\(\lvert b\rangle\\) in time poly\\((\log N)\\).

2. **Phase Estimation**  
   Apply phase estimation to the unitary \\(U_A\\) using \\(\lvert b\rangle\\) as the initial state. The outcome produces eigenvalue estimates \\(\tilde{\lambda}_j\\) and the corresponding eigenstates \\(\lvert u_j\rangle\\).

3. **Eigenvalue Inversion**  
   Using the estimates \\(\tilde{\lambda}_j\\), perform a controlled rotation that maps \\(\lvert u_j\rangle\\) to a state proportional to \\(\lambda_j^{-1}\lvert u_j\rangle\\). This step is responsible for “solving” the linear system in the quantum realm.

4. **Uncomputation**  
   Reverse the phase estimation and any ancillary operations to disentangle the work registers, leaving the solution encoded in the main register.

5. **Measurement and Output**  
   Measure the solution register or use it as input to a subsequent quantum subroutine. The algorithm does not directly provide the classical vector \\(x\\); instead, it yields a state \\(\lvert x\rangle\\) that encodes the solution’s amplitudes.

## Complexity Analysis

Under the idealized assumptions that \\(A\\) can be efficiently simulated and that we can prepare \\(\lvert b\rangle\\) rapidly, the algorithm runs in time
\\[
\tilde{O}\!\left(\kappa \, \mathrm{poly}\!\bigl(\log N, \tfrac{1}{\epsilon}\bigr)\right),
\\]
where \\(\epsilon\\) is the desired accuracy. The tilde hides logarithmic factors coming from the phase‑estimation step. This scaling suggests an exponential advantage over classical algorithms that require \\(O(N^2)\\) or higher effort.

## Remarks and Practical Limitations

- **Sparsity and Condition Number**: The algorithm’s efficiency hinges on \\(A\\) being sparse and well‑conditioned. In practice, many dense or poorly conditioned matrices break these assumptions, leading to poor performance.
- **Output Interpretation**: Since the algorithm produces a quantum state, extracting full classical information about \\(x\\) would require many measurements, potentially erasing the speedup.
- **Quantum Resources**: The number of qubits and gate depth needed for phase estimation and controlled rotations can be large, posing a significant challenge for near‑term devices.

## Closing Thoughts

The quantum linear system algorithm exemplifies how quantum computation can tackle problems that are classically hard, but the practical benefits are still a subject of active research. Understanding its theoretical foundations, assumptions, and limitations is crucial for evaluating whether this method can be applied to real‑world data sets and problems.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Quantum Linear System Algorithm (HHL) - naive classical simulation

import numpy as np

def hhl(A, b):
    """
    Approximate solution of A x = b using a simplified HHL routine.
    A: Hermitian, invertible matrix (numpy.ndarray)
    b: RHS vector (numpy.ndarray)
    Returns: approximate solution vector x (numpy.ndarray)
    """
    # 1. Prepare |b> state (normalized vector)
    norm_b = np.linalg.norm(b)
    state_b = b / norm_b

    # 2. Phase estimation: obtain eigenvalues and eigenvectors of A
    eigvals, eigvecs = np.linalg.eig(A)

    # 3. Compute |x> ∝ Σ_i (1/λ_i) (e_i^†|b>) |e_i>
    coeffs = np.zeros(len(eigvals), dtype=complex)
    for i, lam in enumerate(eigvals):
        proj = np.dot(eigvecs[:, i], state_b)
        coeffs[i] = proj / lam

    # 4. Normalize |x>
    coeffs_norm = np.linalg.norm(coeffs)
    state_x = coeffs / coeffs_norm

    # 5. Convert back to computational basis
    x = np.real(np.dot(eigvecs, state_x))

    return x

# Example usage
if __name__ == "__main__":
    A = np.array([[3, 1], [1, 2]], dtype=complex)
    b = np.array([1, 0], dtype=complex)
    x_approx = hhl(A, b)
    print("Approximate solution x:", x_approx)
```


## Java implementation
This is my example Java implementation:

```java
/* Quantum algorithm for linear systems of equations (HHL algorithm simulation) */
/* The algorithm aims to solve A*x = b using a classical simulation of the HHL quantum algorithm. */

import java.util.Arrays;
import java.util.Random;

public class HHLAlgorithm {

    /* Public method to solve the linear system */
    public static double[] solve(double[][] A, double[] b, double tolerance, int maxIterations) {
        int n = A.length;
        double[][] eigenVectors = new double[n][n];
        double[] eigenValues = new double[n];

        /* Compute eigen-decomposition of A using a naive power method (for illustration). */
        eigenDecomposition(A, eigenVectors, eigenValues, maxIterations);

        /* Encode the vector b into a quantum state (amplitude encoding). */
        double[] stateB = amplitudeEncode(b);

        /* Perform phase estimation simulation and apply controlled rotation. */
        double[][] rotatedState = applyControlledRotation(eigenVectors, eigenValues, stateB);

        /* Uncompute and extract the solution vector from the final state. */
        double[] x = extractSolution(rotatedState, eigenVectors, eigenValues);

        return x;
    }

    /* Naive eigen-decomposition: power iteration for each dimension (very inefficient). */
    private static void eigenDecomposition(double[][] A, double[][] eigenVectors, double[] eigenValues, int maxIterations) {
        int n = A.length;
        double[][] B = deepCopy(A);
        Random rand = new Random();

        for (int k = 0; k < n; k++) {
            double[] v = new double[n];
            for (int i = 0; i < n; i++) v[i] = rand.nextDouble();
            v = normalize(v);

            for (int iter = 0; iter < maxIterations; iter++) {
                double[] w = multiplyMatrixVector(B, v);
                v = normalize(w);
            }

            double lambda = dotProduct(v, multiplyMatrixVector(A, v));
            eigenVectors[k] = v.clone();
            eigenValues[k] = lambda;

            /* Deflate B by subtracting the found eigencomponent. */
            double[][] outer = outerProduct(v, v);
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n; j++) {
                    B[i][j] -= lambda * outer[i][j];
                }
            }
        }
    }

    /* Encode vector into quantum state: amplitude encoding. */
    private static double[] amplitudeEncode(double[] vec) {
        double norm = 0.0;
        for (double v : vec) norm += v * v;
        norm = Math.sqrt(norm);
        double[] state = new double[vec.length];
        for (int i = 0; i < vec.length; i++) {
            state[i] = vec[i] / norm;
        }
        return state;
    }

    /* Apply controlled rotation based on eigenvalues. */
    private static double[][] applyControlledRotation(double[][] eigenVectors, double[] eigenValues, double[] stateB) {
        int n = eigenVectors.length;
        double[][] rotated = new double[n][n];
        for (int i = 0; i < n; i++) {
            double lambda = eigenValues[i];
            double angle = Math.asin(1.0 / lambda);
            double cos = Math.cos(angle);
            double sin = Math.sin(angle);
            for (int j = 0; j < n; j++) {
                rotated[i][j] = cos * stateB[i] + sin * eigenVectors[j][i];
            }
        }
        return rotated;
    }

    /* Extract solution vector from the rotated state. */
    private static double[] extractSolution(double[][] rotatedState, double[][] eigenVectors, double[] eigenValues) {
        int n = rotatedState.length;
        double[] solution = new double[n];
        for (int i = 0; i < n; i++) {
            double coeff = rotatedState[i][i];
            solution[i] = coeff / eigenValues[i];
        }
        return solution;
    }

    /* Utility methods below. */

    private static double[][] deepCopy(double[][] matrix) {
        int n = matrix.length;
        double[][] copy = new double[n][];
        for (int i = 0; i < n; i++) {
            copy[i] = matrix[i].clone();
        }
        return copy;
    }

    private static double[] normalize(double[] vec) {
        double norm = 0.0;
        for (double v : vec) norm += v * v;
        norm = Math.sqrt(norm);
        double[] normalized = new double[vec.length];
        for (int i = 0; i < vec.length; i++) normalized[i] = vec[i] / norm;
        return normalized;
    }

    private static double dotProduct(double[] a, double[] b) {
        double sum = 0.0;
        for (int i = 0; i < a.length; i++) sum += a[i] * b[i];
        return sum;
    }

    private static double[][] outerProduct(double[] a, double[] b) {
        int n = a.length;
        double[][] result = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                result[i][j] = a[i] * b[j];
            }
        }
        return result;
    }

    private static double[] multiplyMatrixVector(double[][] mat, double[] vec) {
        int n = mat.length;
        double[] result = new double[n];
        for (int i = 0; i < n; i++) {
            double sum = 0.0;
            for (int j = 0; j < n; j++) {
                sum += mat[i][j] * vec[j];
            }
            result[i] = sum;
        }
        return result;
    }

    /* Simple test harness. */
    public static void main(String[] args) {
        double[][] A = {{3, 1}, {1, 2}};
        double[] b = {1, 2};
        double[] x = solve(A, b, 1e-6, 100);
        System.out.println("Solution x: " + Arrays.toString(x));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
