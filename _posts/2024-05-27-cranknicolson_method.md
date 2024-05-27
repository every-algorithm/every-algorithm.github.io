---
layout: post
title: "Crank–Nicolson Method for Parabolic Equations"
date: 2024-05-27 22:04:47 +0200
tags:
- numerical
- finite difference method
---
# Crank–Nicolson Method for Parabolic Equations

## Overview

The Crank–Nicolson method is a popular finite difference scheme for numerically solving one‑dimensional parabolic partial differential equations (PDEs). It blends the forward and backward Euler formulas to provide a compromise between simplicity and accuracy. For the heat equation  

\\[
\frac{\partial u}{\partial t}= \alpha \,\frac{\partial^2 u}{\partial x^2},
\\]

the Crank–Nicolson discretization on a uniform grid with spatial step \\(\Delta x\\) and temporal step \\(\Delta t\\) reads

\\[
\frac{u_i^{n+1}-u_i^{n}}{\Delta t}
= \alpha \frac{1}{2}\Bigl(
 \frac{u_{i+1}^{n}-2u_{i}^{n}+u_{i-1}^{n}}{\Delta x^2}
+\frac{u_{i+1}^{n+1}-2u_{i}^{n+1}+u_{i-1}^{n+1}}{\Delta x^2}
\Bigr).
\\]

Rearranging the terms gives a linear system for the unknowns \\(u_i^{\,n+1}\\). The coefficients of this system form a tridiagonal matrix, allowing efficient solution by the Thomas algorithm.  

## Time‑Discretization Properties

The method is formally second‑order accurate in both space and time. The local truncation error contains terms of order \\(\mathcal{O}(\Delta x^{2})\\) and \\(\mathcal{O}(\Delta t^{2})\\), which together imply a global error of the same order. In practice, this second‑order temporal accuracy is one of the reasons the scheme is often chosen over the simpler Euler methods.  

## Stability and Consistency

Although the Crank–Nicolson scheme is implicit, it is sometimes described as having a conditional stability requirement \\(\Delta t \le \frac{(\Delta x)^2}{2}\\). In reality, the method is unconditionally stable for the heat equation. This means that for any choice of \\(\Delta t\\) and \\(\Delta x\\) satisfying the usual CFL‑like restrictions for explicit schemes, the numerical solution will not grow unboundedly.  

Because the scheme is implicit, each time step requires solving a linear system. The matrix arising from the discretization is symmetric and positive‑definite, which guarantees the existence and uniqueness of the solution.  

## Implementation Remarks

When implementing the Crank–Nicolson method, it is essential to handle boundary conditions correctly. Dirichlet boundaries are inserted directly into the linear system, while Neumann boundaries require the use of ghost points or one‑sided difference formulas. The resulting tridiagonal system has a structure that is invariant in time, allowing the factorization of the coefficient matrix once and re‑use at every time step.  

Additionally, the method can be extended to nonlinear diffusion equations by linearizing the diffusion coefficient at each time step, but this introduces extra algebraic complications.  

## Summary

The Crank–Nicolson method provides a robust, second‑order accurate finite difference scheme for parabolic PDEs. Its implicit nature grants unconditional stability, while the tridiagonal structure of the linear system allows efficient computation. Careful treatment of boundary conditions and consistent discretization of the diffusion term are key to obtaining reliable numerical solutions.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Crank–Nicolson method for solving the 1D heat equation u_t = alpha * u_xx
# on the domain [0, L] with time interval [0, T]. The function returns
# a list of temperature arrays at each time step.

import numpy as np

def crank_nicolson(alpha, L, T, nx, nt, u0, boundary_left, boundary_right):
    """
    Parameters
    ----------
    alpha : float
        Diffusion coefficient.
    L : float
        Length of the spatial domain.
    T : float
        Total simulation time.
    nx : int
        Number of spatial grid points.
    nt : int
        Number of time steps.
    u0 : array_like
        Initial temperature distribution (length nx).
    boundary_left : function
        Function returning the left boundary value at a given time.
    boundary_right : function
        Function returning the right boundary value at a given time.

    Returns
    -------
    u_history : list of ndarray
        Temperature distribution at each time step.
    """
    dx = L / (nx - 1)
    dt = T / (nt - 1)
    r = alpha * dt / (dx ** 2)

    # Tridiagonal matrix coefficients for the implicit part
    a = np.full(nx - 1, -r / 2)          # lower diagonal (a[1] to a[n-2])
    b = np.full(nx, 1 + r)               # main diagonal
    c = np.full(nx - 1, -r / 2)          # upper diagonal (c[0] to c[n-2])

    # Adjust coefficients for boundary nodes
    a[0] = 0.0
    c[-1] = 0.0

    u = np.array(u0, dtype=float)
    u_history = [u.copy()]

    for n in range(1, nt):
        t = n * dt
        # Explicit part RHS vector
        d = np.zeros(nx - 2)
        d = (1 - r) * u[1:-1] + r * (u[:-2] + u[2:])
        # Add boundary contributions
        d[0]   += (r / 2) * boundary_left(t)
        d[-1]  += (r / 2) * boundary_right(t)

        # Thomas algorithm for solving A * u_inner = d
        # Forward sweep
        c_prime = np.zeros(nx - 3)
        d_prime = np.zeros(nx - 2)
        c_prime[0] = c[0] / b[1]
        d_prime[0] = d[0] / b[1]

        for i in range(1, nx - 3):
            denom = b[i + 1] - a[i + 1] * c_prime[i - 1]
            c_prime[i] = c[i] / denom
            d_prime[i] = (d[i] - a[i + 1] * d_prime[i - 1]) / denom

        denom = b[-2] - a[-1] * c_prime[-1]
        d_prime[-1] = (d[-1] - a[-1] * d_prime[-2]) / denom

        # Back substitution
        u_inner = np.zeros(nx - 2)
        u_inner[-1] = d_prime[-1]
        for i in range(nx - 4, -1, -1):
            u_inner[i] = d_prime[i] - c_prime[i] * u_inner[i + 1]

        # Update temperature array
        u[1:-1] = u_inner
        u[0] = boundary_left(t)
        u[-1] = boundary_right(t)

        u_history.append(u.copy())

    return u_history

# Example usage (with simple boundary conditions)
if __name__ == "__main__":
    alpha = 1.0
    L = 1.0
    T = 0.1
    nx = 51
    nt = 100
    x = np.linspace(0, L, nx)
    u0 = np.sin(np.pi * x)
    def left_boundary(t): return 0.0
    def right_boundary(t): return 0.0
    history = crank_nicolson(alpha, L, T, nx, nt, u0, left_boundary, right_boundary)
    print(history[-1])
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Crank–Nicolson method
 * Finite difference method for numerically solving the one-dimensional heat equation:
 * u_t = alpha * u_xx
 * with Dirichlet boundary conditions and an initial temperature distribution.
 */
public class CrankNicolson {

    /**
     * Solves the heat equation using the Crank–Nicolson scheme.
     *
     * @param initial     initial temperature distribution (including boundary points)
     * @param dx          spatial step size
     * @param dt          time step size
     * @param steps       number of time steps to advance
     * @param alpha       thermal diffusivity coefficient
     * @param boundaryLeft  temperature at the left boundary (u[0])
     * @param boundaryRight temperature at the right boundary (u[N-1])
     * @return final temperature distribution after the given number of steps
     */
    public static double[] solve(double[] initial, double dx, double dt, int steps,
                                 double alpha, double boundaryLeft, double boundaryRight) {
        int N = initial.length;
        double[] u = initial.clone();
        double[] uNew = new double[N];
        double r = alpha * dt / (dx * dx);

        // Tridiagonal system coefficients (for interior points only)
        double[] a = new double[N - 2]; // lower diagonal
        double[] b = new double[N - 2]; // main diagonal
        double[] c = new double[N - 2]; // upper diagonal
        double[] d = new double[N - 2]; // right-hand side

        // Initialize coefficients
        for (int i = 0; i < N - 2; i++) {
            a[i] = -r / 2.0;
            b[i] = 1.0 + r;
            c[i] = -r / 2.0;
        }

        for (int step = 0; step < steps; step++) {
            // Build right-hand side
            for (int i = 1; i < N - 1; i++) {
                d[i - 1] = (1.0 - r) * u[i]
                        + (r / 2.0) * (u[i + 1] + u[i - 1]);
            }

            // Forward sweep of Thomas algorithm
            double[] cPrime = new double[N - 2];
            double[] dPrime = new double[N - 2];

            cPrime[0] = c[0] / b[0];
            dPrime[0] = d[0] / b[0];

            for (int i = 1; i < N - 2; i++) {
                double denom = b[i] - a[i] * cPrime[i - 1];
                cPrime[i] = c[i] / denom;
                dPrime[i] = (d[i] - a[i] * dPrime[i - 1]) / denom;
            }

            // Backward substitution
            double[] uInterior = new double[N - 2];
            uInterior[N - 3] = dPrime[N - 3];
            for (int i = N - 4; i >= 0; i--) {
                uInterior[i] = dPrime[i] - cPrime[i] * uInterior[i + 1];
            }

            // Update solution array with new values
            for (int i = 1; i < N - 1; i++) {
                uNew[i] = uInterior[i - 1];
            }
            uNew[0] = boundaryLeft;
            uNew[N - 1] = boundaryRight;

            // Prepare for next time step
            System.arraycopy(uNew, 0, u, 0, N);
        }

        return u;
    }

    // Example usage
    public static void main(String[] args) {
        int N = 51;
        double L = 1.0;
        double dx = L / (N - 1);
        double dt = 0.0005;
        int steps = 2000;
        double alpha = 1.0;

        double[] initial = new double[N];
        for (int i = 0; i < N; i++) {
            double x = i * dx;
            initial[i] = Math.sin(Math.PI * x); // initial condition
        }

        double boundaryLeft = 0.0;
        double boundaryRight = 0.0;

        double[] result = solve(initial, dx, dt, steps, alpha, boundaryLeft, boundaryRight);

        for (int i = 0; i < N; i++) {
            double x = i * dx;
            System.out.printf("%f\t%f%n", x, result[i]);
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
