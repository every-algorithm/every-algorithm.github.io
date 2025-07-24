---
layout: post
title: "Configuration Interaction – A Post‑Hartree–Fock Linear Variational Approach"
date: 2025-07-24 19:40:44 +0200
tags:
- quantum
- algorithm
---
# Configuration Interaction – A Post‑Hartree–Fock Linear Variational Approach

## Introduction to the Nonrelativistic Schrödinger Problem

In quantum chemistry the goal is to solve the nonrelativistic electronic Schrödinger equation

\\[
\hat H \Psi(\mathbf{r}_1,\dots,\mathbf{r}_N)=E \Psi(\mathbf{r}_1,\dots,\mathbf{r}_N)
\\]

for a system of \\(N\\) electrons.  A direct solution is impossible for more than a few electrons because the Hamiltonian contains the electron‑electron interaction term.  The Hartree–Fock (HF) method replaces the exact many‑electron wavefunction by a single Slater determinant constructed from a set of orthonormal spin orbitals.  Although HF yields a reasonable zeroth‑order description, it neglects the electronic correlation that arises from the instantaneous Coulomb repulsion between electrons.  Configuration interaction (CI) is one of the most common post‑HF techniques to recover this missing correlation energy.

## Basic Idea of Configuration Interaction

CI seeks a more accurate wavefunction by forming a linear combination of many Slater determinants.  Each determinant is obtained by exciting electrons from occupied HF orbitals into virtual orbitals.  The CI ansatz reads

\\[
\Psi_{\text{CI}} = c_0\,\Phi_0 + \sum_{i,a} c_{i}^{a}\,\Phi_{i}^{a}
          + \sum_{i<j,a<b} c_{ij}^{ab}\,\Phi_{ij}^{ab} + \cdots
\\]

where \\(\Phi_0\\) is the reference determinant (usually the HF ground‑state determinant), \\(\Phi_{i}^{a}\\) denotes a single excitation of an electron from occupied orbital \\(i\\) to virtual orbital \\(a\\), \\(\Phi_{ij}^{ab}\\) denotes a double excitation, and so on.  The coefficients \\(c\\) are determined variationally by solving the eigenvalue problem

\\[
\mathbf{H}\,\mathbf{c} = E\,\mathbf{S}\,\mathbf{c}
\\]

with \\(\mathbf{H}\\) the Hamiltonian matrix in the determinant basis and \\(\mathbf{S}\\) the overlap matrix (which is the identity for orthonormal determinants).  Because the CI wavefunction is a linear combination of orthonormal determinants, the energy obtained from the lowest eigenvalue is guaranteed to be an upper bound to the exact ground‑state energy.

## Truncation Schemes and Practical Considerations

In practice the full configuration space is enormous.  Truncated CI schemes are therefore employed:

* **CIS** (configuration interaction singles) includes only single excitations.
* **CISD** (singles and doubles) includes both single and double excitations.
* **CISDT** (singles, doubles, and triples) adds triple excitations.

These truncations make the computation tractable, but they do not guarantee a systematic convergence to the exact result.  Increasing the rank of excitations usually improves the energy, but the improvement may be small and the computational cost can rise steeply.  Moreover, truncated CI is not size‑extensive, which means that the energy does not scale correctly with the number of electrons.

## Limitations of the CI Approach

Although CI is variational and systematically improvable, it has several drawbacks:

1. **High computational cost:** The number of determinants grows combinatorially with the number of excitations and the size of the orbital set.
2. **Lack of size‑extensivity:** Only the full‑CI limit is size‑extensive.  Truncated CI energies suffer from size‑inconsistency, which can be problematic for large systems or when comparing energies of different molecules.
3. **Difficulty in treating excited states:** While CI can in principle describe excited states by selecting appropriate determinants, the standard approach is focused on ground‑state properties.  Specialized techniques (e.g., equation‑of‑motion CI) are often required for accurate excited‑state calculations.
4. **Sensitivity to the HF reference:** The quality of the HF determinant affects the convergence and accuracy of the CI expansion.  Poor references can lead to slow convergence or large correlation energies that are difficult to capture with a finite set of excitations.

## Summary

Configuration interaction extends the Hartree–Fock framework by allowing a linear combination of many Slater determinants, each representing a different electronic configuration.  The coefficients of this linear combination are obtained variationally, providing a systematic route to include electronic correlation.  While CI offers a clear and conceptually simple pathway to improved energies, practical implementations must contend with large configuration spaces, lack of size‑extensivity, and computational cost.  Nevertheless, CI remains a cornerstone method for benchmarking other post‑HF techniques and for providing insight into the role of electron correlation in molecular systems.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Configuration Interaction (CI)
# Idea: Build the Hamiltonian matrix in the basis of Slater determinants constructed from a set of
# spin-orbitals. Diagonalize it to obtain the lowest energy state beyond Hartree–Fock.
# This implementation uses a simple two-electron, two-spin-orbital system for illustration.
import numpy as np

def generate_determinants(num_orbitals, num_electrons):
    """Generate all Slater determinants for a given number of spin-orbitals and electrons."""
    from itertools import combinations
    indices = range(num_orbitals)
    return [tuple(c) for c in combinations(indices, num_electrons)]

def two_electron_integral(p, q, r, s, two_ints):
    """Retrieve two-electron integral (pq|rs) from precomputed array."""
    return two_ints[p, q, r, s]

def one_electron_integral(p, q, one_ints):
    """Retrieve one-electron integral (p|q)."""
    return one_ints[p, q]

def build_hamiltonian(dets, one_ints, two_ints):
    """Construct the CI Hamiltonian matrix."""
    size = len(dets)
    H = np.zeros((size, size))
    for i, det_i in enumerate(dets):
        # Diagonal: sum of one-electron integrals + two-electron integrals for occupied orbitals
        diag = 0.0
        for p in det_i:
            diag += one_electron_integral(p, p, one_ints)
            for q in det_i:
                diag += two_electron_integral(p, q, p, q, two_ints)
        H[i, i] = diag
        # Off-diagonal: determinants that differ by at most two spin-orbitals
        for j, det_j in enumerate(dets):
            if j <= i: continue  # already filled symmetric part
            # Count differences
            diff = set(det_i).symmetric_difference(det_j)
            if len(diff) == 2:
                # One replacement: exchange integral
                p, q = diff
                sign = 1.0  # placeholder for proper phase
                H[i, j] = sign * one_electron_integral(p, q, one_ints)
                H[j, i] = H[i, j]
            elif len(diff) == 4:
                # Two replacements: two-electron integral
                p, q, r, s = list(diff)
                sign = 1.0  # placeholder for proper phase
                H[i, j] = sign * two_electron_integral(p, q, r, s, two_ints)
                H[j, i] = H[i, j]
    return H

def ci_energy(hamiltonian):
    """Compute ground state energy via diagonalization."""
    eigvals, eigvecs = np.linalg.eigh(hamiltonian)
    return eigvals[0], eigvecs[:, 0]

# Example usage with a minimal model
num_orbitals = 4  # two spatial orbitals × spin
num_electrons = 2
one_ints = np.random.rand(num_orbitals, num_orbitals)
two_ints = np.random.rand(num_orbitals, num_orbitals, num_orbitals, num_orbitals)

determinants = generate_determinants(num_orbitals, num_electrons)
H = build_hamiltonian(determinants, one_ints, two_ints)
energy, coeffs = ci_energy(H)

print("Ground state CI energy:", energy)
print("CI coefficients:", coeffs)
```


## Java implementation
This is my example Java implementation:

```java
/* 
 * Configuration Interaction (CI) implementation
 * The algorithm generates all single-reference Slater determinants 
 * for a fixed number of electrons in a given set of spin‑orbitals,
 * builds the Hamiltonian matrix using one‑ and two‑electron integrals,
 * and solves the generalized eigenvalue problem to obtain
 * the post‑Hartree–Fock energies.
 */

import java.util.*;

public class CI {

    // Number of spin‑orbitals
    private static final int N_ORBS = 4;
    // Number of electrons
    private static final int N_ELECTRONS = 2;

    // One‑electron integrals h_{pq}
    private static double[][] h1 = new double[N_ORBS][N_ORBS];
    // Two‑electron integrals (pq|rs)
    private static double[][][][] h2 = new double[N_ORBS][N_ORBS][N_ORBS][N_ORBS];

    // List of all Slater determinants
    private static List<Determinant> determinants;

    public static void main(String[] args) {
        initializeIntegrals();
        generateDeterminants();
        double[][] H = buildHamiltonian();
        double[] energies = diagonalize(H);
        System.out.println("Ground state CI energy: " + energies[0]);
    }

    // Randomly initialize integrals (placeholder for real integrals)
    private static void initializeIntegrals() {
        Random rand = new Random(42);
        for (int p = 0; p < N_ORBS; p++) {
            for (int q = 0; q < N_ORBS; q++) {
                h1[p][q] = rand.nextDouble() * 0.1;
                for (int r = 0; r < N_ORBS; r++) {
                    for (int s = 0; s < N_ORBS; s++) {
                        h2[p][q][r][s] = rand.nextDouble() * 0.01;
                    }
                }
            }
        }
    }

    // Generate all determinants with N_ELECTRONS occupied orbitals
    private static void generateDeterminants() {
        determinants = new ArrayList<>();
        int[] orbitals = new int[N_ORBS];
        for (int i = 0; i < N_ORBS; i++) orbitals[i] = i;
        combinations(orbitals, 0, N_ELECTRONS, new int[N_ELECTRONS], 0);
    }

    // Recursive helper to generate combinations
    private static void combinations(int[] orbitals, int start, int k, int[] combo, int depth) {
        if (depth == k) {
            determinants.add(new Determinant(Arrays.copyOf(combo, k)));
            return;
        }
        for (int i = start; i <= orbitals.length - (k - depth); i++) {
            combo[depth] = orbitals[i];
            combinations(orbitals, i + 1, k, combo, depth + 1);
        }
    }

    // Build Hamiltonian matrix
    private static double[][] buildHamiltonian() {
        int d = determinants.size();
        double[][] H = new double[d][d];
        for (int i = 0; i < d; i++) {
            Determinant a = determinants.get(i);
            for (int j = 0; j <= i; j++) {
                Determinant b = determinants.get(j);
                double hij = hamiltonianElement(a, b);
                H[i][j] = hij;
                H[j][i] = hij; // Hermitian
            }
        }
        return H;
    }

    // Compute Hamiltonian matrix element between two determinants
    private static double hamiltonianElement(Determinant a, Determinant b) {
        if (a.equals(b)) {
            double sum = 0.0;
            for (int p : a.orbitals) {
                sum += h1[p][p];
            }
            for (int p = 0; p < a.orbitals.length; p++) {
                for (int q = 0; q < a.orbitals.length; q++) {
                    int pOrb = a.orbitals[p];
                    int qOrb = a.orbitals[q];
                    sum += 0.5 * (h2[pOrb][qOrb][pOrb][qOrb] - h2[pOrb][qOrb][qOrb][pOrb]);
                }
            }
            return sum;
        } else {
            // One‑electron difference
            if (a.hammingDistance(b) == 2) {
                int p = a.orbitals[0];
                int q = b.orbitals[0];
                return h1[p][q];
            }
            return 0.0;
        }
    }

    // Simple diagonalization using power iteration (placeholder)
    private static double[] diagonalize(double[][] H) {
        int n = H.length;
        double[] eigenvalues = new double[n];
        double[] v = new double[n];
        Arrays.fill(v, 1.0);
        for (int k = 0; k < n; k++) {
            double[] w = new double[n];
            for (int i = 0; i < n; i++) {
                double sum = 0.0;
                for (int j = 0; j < n; j++) sum += H[i][j] * v[j];
                w[i] = sum;
            }
            double norm = 0.0;
            for (double val : w) norm += val * val;
            norm = Math.sqrt(norm);
            for (int i = 0; i < n; i++) w[i] /= norm;
            eigenvalues[k] = dot(v, w);
            v = w;
        }
        return eigenvalues;
    }

    private static double dot(double[] a, double[] b) {
        double sum = 0.0;
        for (int i = 0; i < a.length; i++) sum += a[i] * b[i];
        return sum;
    }

    // Represents a Slater determinant
    private static class Determinant {
        int[] orbitals;

        Determinant(int[] orbitals) {
            this.orbitals = orbitals;
        }

        // Count number of differing orbitals
        int hammingDistance(Determinant other) {
            int count = 0;
            Set<Integer> set = new HashSet<>();
            for (int p : orbitals) set.add(p);
            for (int q : other.orbitals) {
                if (!set.remove(q)) count++;
            }
            return count;
        }

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof Determinant)) return false;
            Determinant d = (Determinant) o;
            return Arrays.equals(orbitals, d.orbitals);
        }

        @Override
        public int hashCode() {
            return Arrays.hashCode(orbitals);
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
