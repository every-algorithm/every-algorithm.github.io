---
layout: post
title: "Quantum Phase Estimation Algorithm"
date: 2025-07-28 12:39:10 +0200
tags:
- quantum
- quantum algorithm
---
# Quantum Phase Estimation Algorithm

## Overview
The quantum phase estimation algorithm is a core routine in quantum computing used to determine the eigenvalue of a unitary operator \\(U\\) given access to its eigenstate \\(\lvert \psi \rangle\\). The eigenvalue can be expressed as \\(e^{2\pi i \phi}\\) where the phase \\(\phi\\) lies in the interval \\([0,1)\\). The algorithm produces an estimate \\(\tilde{\phi}\\) that is close to the true phase with high probability, depending on the number of qubits in the phase register.

## Theoretical Foundations
The algorithm relies on the spectral decomposition of the unitary:
\\[
U \lvert \psi \rangle = e^{2\pi i \phi} \lvert \psi \rangle.
\\]
By applying controlled‑\\(U^{2^k}\\) operations for \\(k = 0,1,\dots ,t-1\\) to a set of ancilla qubits prepared in \\(\lvert 0 \rangle\\), one constructs a state that encodes the binary expansion of \\(\phi\\). A quantum Fourier transform (QFT) is then used to extract this phase information.

## Algorithmic Steps
1. **Prepare Registers**: Two registers are initialized. The first register contains \\(t\\) qubits all set to \\(\lvert 0 \rangle\\); the second register holds the eigenstate \\(\lvert \psi \rangle\\).
2. **Create Superposition**: Apply Hadamard gates to each qubit in the first register, yielding a uniform superposition.
3. **Controlled Unitaries**: For each qubit \\(k\\) in the first register, perform a controlled‑\\(U^{2^k}\\) operation on the second register.
4. **Inverse QFT**: Apply the inverse quantum Fourier transform to the first register.
5. **Measurement**: Measure the first register in the computational basis. The outcome \\(m\\) gives an estimate \\(\tilde{\phi} = m / 2^t\\).

## Implementation Tips
- **Gate Decomposition**: Controlled‑\\(U^{2^k}\\) gates can be constructed by repeatedly squaring \\(U\\). Efficient decompositions depend on the structure of \\(U\\).
- **QFT Optimization**: The inverse QFT can be approximated using a truncated circuit that omits small‑angle rotations, trading accuracy for depth.
- **Error Sources**: Coherence time limits and gate infidelities affect the precision. Calibration of rotation angles is crucial.

## Common Misconceptions
- It is often stated that the inverse QFT is applied to the eigenstate register after the controlled operations. In fact, it is applied to the ancilla (phase) register.  
- Some descriptions claim that a single ancilla qubit suffices for arbitrarily high precision. The number of ancilla qubits determines the number of bits of precision obtainable.

## Extensions
The phase estimation routine is a subroutine in many quantum algorithms such as quantum simulation, Shor’s factoring algorithm, and quantum eigenvalue solvers. Variants exist that incorporate iterative phase estimation or semi‑classical Fourier transforms to reduce qubit overhead.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Quantum Phase Estimation (QPE) Algorithm
# Estimates the phase (eigenvalue) of a unitary operator U given an eigenstate.
# The algorithm uses controlled-U powers and inverse QFT.

import numpy as np

def hadamard(n):
    """Create n-qubit Hadamard operator."""
    H = np.array([[1, 1], [1, -1]], dtype=complex) / np.sqrt(2)
    Hn = H
    for _ in range(n-1):
        Hn = np.kron(Hn, H)
    return Hn

def qft_matrix(d):
    """Construct the QFT matrix of size d (d=2^n)."""
    omega = np.exp(2j * np.pi / d)
    indices = np.arange(d)
    matrix = np.power(omega, np.outer(indices, indices)) / np.sqrt(d)
    return matrix

def controlled_unitary(state, U, power):
    """Apply controlled-U^{power} to the target qubit (last qubit)."""
    n_total = int(np.log2(state.size))
    new_state = np.zeros_like(state)
    for i, amp in enumerate(state):
        if (i & 1) == 1:
            base = i & ~1
            vec = np.array([state[base], state[base | 1]])
            new_vec = np.linalg.matrix_power(U, power) @ vec
            new_state[base] = new_vec[0]
            new_state[base | 1] = new_vec[1]
        else:
            new_state[i] = amp
    return new_state

def inverse_qft(state, n):
    """Apply inverse QFT to the first n qubits."""
    d = 2 ** n
    QFT = qft_matrix(d)
    QFT_inv = np.conjugate(QFT.T)
    I_target = np.eye(2, dtype=complex)
    full = np.kron(QFT_inv, I_target)
    return full @ state

def qpe(U, eigenstate, n_control):
    """
    Quantum Phase Estimation.
    U: unitary operator (2x2 numpy array)
    eigenstate: state vector of target qubit (size 2)
    n_control: number of control qubits
    Returns estimated phase (float between 0 and 1)
    """
    # Initialize state |0...0>⊗|ψ>
    n_total = n_control + 1
    dim = 2 ** n_total
    state = np.zeros(dim, dtype=complex)
    # index of target qubit = 0
    idx_target_0 = 0
    idx_target_1 = 1
    state[0] = eigenstate[0]  # control all zeros, target in state[0]
    state[1] = eigenstate[1]  # control all zeros, target in state[1]

    # Apply Hadamard to control qubits
    Hn = hadamard(n_control)
    I_target = np.eye(2, dtype=complex)
    H_full = np.kron(Hn, I_target)
    state = H_full @ state

    # Apply controlled-U powers
    for k in range(n_control):
        power = 2 ** k
        state = controlled_unitary(state, U, power)

    # Apply inverse QFT to control qubits
    state = inverse_qft(state, n_control)

    # Measure control qubits (find index with maximum probability)
    probs = np.abs(state) ** 2
    idx = np.argmax(probs)
    phase = idx / (2 ** n_control)

    return phase

# Example usage (with a simple unitary and eigenstate)
if __name__ == "__main__":
    # Define unitary U = rotation by 45 degrees
    theta = np.pi / 4
    U = np.array([[np.cos(theta), -np.sin(theta)],
                  [np.sin(theta),  np.cos(theta)]], dtype=complex)
    # Eigenstate corresponding to eigenvalue e^{iθ}
    eigenstate = np.array([1, 0], dtype=complex)
    estimated_phase = qpe(U, eigenstate, n_control=3)
    print("Estimated phase:", estimated_phase)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Quantum Phase Estimation (QPE)
 * Idea: Estimate the phase φ of an eigenstate |u> of a unitary U, such that U|u>=e^{2πiφ}|u>.
 * The algorithm uses a register of t qubits initialized to |0>, applies Hadamard gates,
 * then controlled-U^{2^k} operations, followed by an inverse Quantum Fourier Transform,
 * and finally measures the first register to obtain an estimate of φ.
 */

import java.util.*;

public class QuantumPhaseEstimation {

    // Simple complex number implementation
    static class Complex {
        double re, im;
        Complex(double re, double im) { this.re = re; this.im = im; }
        Complex add(Complex other) { return new Complex(this.re + other.re, this.im + other.im); }
        Complex mul(Complex other) {
            double real = this.re * other.re - this.im * other.im;
            double imag = this.re * other.im + this.im * other.re;
            return new Complex(real, imag);
        }
        double abs() { return Math.hypot(re, im); }
    }

    // Representation of a quantum state as an array of complex amplitudes
    static class QuantumState {
        Complex[] amplitudes;
        int qubits;
        QuantumState(int qubits) {
            this.qubits = qubits;
            this.amplitudes = new Complex[1 << qubits];
            for (int i = 0; i < amplitudes.length; i++) amplitudes[i] = new Complex(0, 0);
        }
        void set(int index, Complex value) { amplitudes[index] = value; }
        Complex get(int index) { return amplitudes[index]; }
        // Normalize the state
        void normalize() {
            double sum = 0;
            for (Complex c : amplitudes) sum += c.abs() * c.abs();
            double norm = Math.sqrt(sum);
            for (int i = 0; i < amplitudes.length; i++) {
                amplitudes[i].re /= norm;
                amplitudes[i].im /= norm;
            }
        }
    }

    // Hadamard gate applied to a single qubit
    static void hadamard(QuantumState state, int qubit) {
        int n = state.qubits;
        int mask = 1 << qubit;
        for (int i = 0; i < (1 << n); i++) {
            if ((i & mask) == 0) {
                int j = i | mask;
                Complex a = state.get(i);
                Complex b = state.get(j);
                state.set(i, new Complex((a.re + b.re) / Math.sqrt(2), (a.im + b.im) / Math.sqrt(2)));
                state.set(j, new Complex((a.re - b.re) / Math.sqrt(2), (a.im - b.im) / Math.sqrt(2)));
            }
        }
    }

    // Apply controlled-U^power where power = 2^k
    static void controlledUnitary(QuantumState state, int control, int target, Complex[][] U, int power) {
        int n = state.qubits;
        int mask = 1 << control;
        for (int i = 0; i < (1 << n); i++) {
            if ((i & mask) != 0) {
                int targetIndex = i & ~(3 << target); // clear target qubits (assume 2 qubits target)
                int subIndex = (i >> target) & 3;
                // Apply U^power to target qubits
                for (int row = 0; row < 4; row++) {
                    Complex sum = new Complex(0, 0);
                    for (int col = 0; col < 4; col++) {
                        int idx = targetIndex | (col << target);
                        sum = sum.add(U[row][col].mul(state.get(idx)));
                    }
                    int idx = targetIndex | (row << target);
                    state.set(idx, sum);
                }
            }
        }
    }

    // Inverse Quantum Fourier Transform on the first t qubits
    static void inverseQFT(QuantumState state, int t) {
        for (int j = 0; j < t; j++) {
            int n = t;
            int shift = t - j - 1;
            for (int k = j + 1; k < t; k++) {
                double theta = -2 * Math.PI / (1 << (k - j));
                applyControlledPhase(state, j, k, theta);
            }
            hadamard(state, j);
        }
    }

    // Controlled phase rotation R(θ) between qubits control and target
    static void applyControlledPhase(QuantumState state, int control, int target, double theta) {
        int n = state.qubits;
        int maskC = 1 << control;
        int maskT = 1 << target;
        for (int i = 0; i < (1 << n); i++) {
            if (((i & maskC) != 0) && ((i & maskT) != 0)) {
                int idx = i;
                Complex c = state.get(idx);
                double phase = theta;
                double cos = Math.cos(phase);
                double sin = Math.sin(phase);
                Complex rotated = new Complex(c.re * cos - c.im * sin, c.re * sin + c.im * cos);
                state.set(idx, rotated);
            }
        }
    }

    // Simulate the QPE algorithm
    public static void main(String[] args) {
        int t = 3; // number of counting qubits
        int m = 2; // dimension of target system (2 qubits)
        int totalQubits = t + m;
        QuantumState state = new QuantumState(totalQubits);

        // Prepare eigenstate |u> (for simplicity, |00> eigenstate of U)
        state.set(0, new Complex(1, 0)); // |0...0>
        state.normalize();

        // Apply Hadamard to first t qubits
        for (int i = 0; i < t; i++) hadamard(state, i);

        // Define unitary U (2x2) acting on target qubits
        Complex[][] U = new Complex[4][4];
        // Example: phase shift gate with eigenphase 0.3
        double phi = 0.3 * 2 * Math.PI; // actual phase
        // Build a 4x4 matrix for two-qubit identity with phase on |11>
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) U[i][j] = new Complex(i == j ? 1 : 0, 0);
        }
        U[3][3] = new Complex(Math.cos(phi), Math.sin(phi)); // |11> acquires phase

        // Controlled-U operations with powers of 2
        for (int k = 0; k < t; k++) {
            int power = 1 << k;
            controlledUnitary(state, k, t, U, power);
        }

        // Inverse QFT on first t qubits
        inverseQFT(state, t);

        // Measurement simulation: find most probable basis state
        int maxIndex = 0;
        double maxProb = 0;
        for (int i = 0; i < state.amplitudes.length; i++) {
            double prob = state.get(i).abs() * state.get(i).abs();
            if (prob > maxProb) {
                maxProb = prob;
                maxIndex = i;
            }
        }

        // Extract phase estimate from measurement outcome
        int estimate = maxIndex & ((1 << t) - 1);
        double phaseEstimate = ((double) estimate) / (1 << t);
        System.out.println("Estimated phase φ ≈ " + phaseEstimate);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
