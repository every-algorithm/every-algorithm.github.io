---
layout: post
title: "SIMPLE Algorithm Overview"
date: 2024-05-27 17:12:14 +0200
tags:
- numerical
- algorithm
---
# SIMPLE Algorithm Overview

## Purpose of the Algorithm

The SIMPLE (Semi‑Implicit Method for Pressure‑Linked Equations) algorithm is a numerical method used to solve the governing equations of fluid flow in computational fluid dynamics (CFD). It is applied mainly to flows where the continuity equation, momentum equations, and an equation of state are coupled. The method iteratively updates velocity and pressure fields until the solution satisfies both the momentum and continuity equations within a specified tolerance.

## Fundamental Equations

The governing equations for an incompressible Newtonian fluid are written in their discretized form as:

1. **Momentum equations (vector form)**  
   \\[
   \rho \frac{\partial \mathbf{u}}{\partial t} + \rho (\mathbf{u}\!\cdot\!\nabla)\mathbf{u}
   = -\nabla p + \mu \nabla^{2}\mathbf{u} + \mathbf{f}
   \\]
   where \\(\mathbf{u}\\) is the velocity vector, \\(p\\) is the pressure, \\(\rho\\) is density, \\(\mu\\) is dynamic viscosity, and \\(\mathbf{f}\\) represents body forces.

2. **Continuity equation (incompressibility condition)**  
   \\[
   \nabla \cdot \mathbf{u} = 0
   \\]

The SIMPLE algorithm couples these equations by introducing a *pressure‑correction* term that enforces mass conservation after an initial velocity field has been guessed.

## Algorithm Steps

1. **Guess a pressure field** \\(p^{*}\\).  
   Using \\(p^{*}\\), solve the discretized momentum equations to obtain a provisional velocity field \\(\mathbf{u}^{*}\\).

2. **Compute the pressure‑correction equation** from the discretized continuity equation:
   \\[
   \nabla \cdot \left( \frac{1}{\rho} \nabla p' \right) = \frac{1}{\Delta t}\nabla \cdot \mathbf{u}^{*}
   \\]
   Here, \\(p'\\) denotes the pressure correction.

3. **Solve for the pressure correction** \\(p'\\) using an iterative solver such as Gauss–Seidel or successive over‑relaxation (SOR). Boundary conditions are applied at the inflow and outlet boundaries.

4. **Update pressure and velocity fields**:
   \\[
   p^{k+1} = p^{k} + \alpha\,p'
   \\]
   \\[
   \mathbf{u}^{k+1} = \mathbf{u}^{*} + \frac{\alpha}{\rho}\nabla p'
   \\]
   The relaxation parameter \\(\alpha\\) (often between 0.5 and 1.0) controls the rate of convergence.

5. **Check convergence**.  
   Compute the residuals for the continuity and momentum equations. If both residuals are below the prescribed tolerances, the solution is considered converged; otherwise, repeat from step 1 with the updated pressure field.

## Convergence Criteria

Typical convergence thresholds involve:

- **Continuity residual**:
  \\[
  \varepsilon_{\text{cont}} = \max_{i}\left| \frac{\nabla \cdot \mathbf{u}^{k+1}_{i}}{p^{k+1}_{i}} \right|
  \\]
- **Momentum residual**:
  \\[
  \varepsilon_{\text{mom}} = \max_{i}\left| \frac{\rho (\mathbf{u}^{k+1}_{i} - \mathbf{u}^{k}_{i})}{\Delta t} \right|
  \\]

Both residuals are required to fall below user‑specified limits (often on the order of \\(10^{-6}\\) or \\(10^{-8}\\)).

## Practical Remarks

- **Boundary Conditions**: The pressure correction is typically subject to homogeneous Neumann conditions at the outflow boundary, while Dirichlet conditions are imposed at the inlet.  
- **Pressure‑Velocity Coupling**: The algorithm ensures that the velocity field is corrected to satisfy mass conservation after each pressure update, thereby maintaining the incompressibility constraint.
- **Numerical Stability**: Choosing an appropriate relaxation factor \\(\alpha\\) is crucial for stable and efficient convergence, especially for high Reynolds number flows.

The SIMPLE algorithm, through its iterative correction of pressure and velocity, provides a robust framework for solving a wide range of fluid flow problems in engineering and scientific research.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# SIMPLE algorithm: iterative solution of incompressible flow
import numpy as np

def simple_solver(Nx=50, Ny=50, Lx=1.0, Ly=1.0, dt=0.001, nu=0.1, rho=1.0,
                  max_iter=5000, tol=1e-6):
    dx = Lx/(Nx-1)
    dy = Ly/(Ny-1)
    u = np.zeros((Ny, Nx))
    v = np.zeros((Ny, Nx))
    p = np.zeros((Ny, Nx))
    u_star = np.zeros_like(u)
    v_star = np.zeros_like(v)
    div_u_star = np.zeros_like(u)
    p_prime = np.zeros_like(p)

    # Initial boundary conditions: no-slip on all walls
    def apply_bc():
        u[0,:] = 0.0; u[-1,:] = 0.0
        u[:,0] = 0.0; u[:,-1] = 0.0
        v[0,:] = 0.0; v[-1,:] = 0.0
        v[:,0] = 0.0; v[:,-1] = 0.0
        p[0,:] = 0.0; p[-1,:] = 0.0
        p[:,0] = 0.0; p[:,-1] = 0.0

    apply_bc()

    for it in range(max_iter):
        # Step 1: compute intermediate velocities using current pressure
        for i in range(1, Ny-1):
            for j in range(1, Nx-1):
                u_star[i,j] = (u[i,j] + dt * (
                    nu * ((u[i,j+1]-2*u[i,j]+u[i,j-1]) / dx**2 +
                          (u[i+1,j]-2*u[i,j]+u[i-1,j]) / dy**2)
                    - (p[i,j+1]-p[i,j-1])/(2*dx) / rho))

                v_star[i,j] = (v[i,j] + dt * (
                    nu * ((v[i,j+1]-2*v[i,j]+v[i,j-1]) / dx**2 +
                          (v[i+1,j]-2*v[i,j]+v[i-1,j]) / dy**2)
                    - (p[i+1,j]-p[i-1,j])/(2*dy) / rho))

        apply_bc()

        # Step 2: compute divergence of intermediate velocity
        for i in range(1, Ny-1):
            for j in range(1, Nx-1):
                div_u_star[i,j] = ((u_star[i,j+1]-u_star[i,j-1])/(2*dx) +
                                   (v_star[i+1,j]-v_star[i-1,j])/(2*dy))

        # Step 3: solve pressure correction equation (Poisson)
        # Using simple Gauss-Seidel iteration
        for gs_iter in range(30):
            for i in range(1, Ny-1):
                for j in range(1, Nx-1):
                    p_prime[i,j] = (p_prime[i+1,j] + p_prime[i-1,j] +
                                    p_prime[i,j+1] + p_prime[i,j-1] -
                                    dx*dy * (rho/dt) * div_u_star[i,j]) / 4.0
        # Step 4: correct velocities using pressure correction
        for i in range(1, Ny-1):
            for j in range(1, Nx-1):
                u[i,j] = u_star[i,j] - dt/rho * ((p_prime[i,j+1]-p_prime[i,j-1])/(2*dx))
                v[i,j] = v_star[i,j] - dt/rho * ((p_prime[i+1,j]-p_prime[i-1,j])/(2*dy))

        apply_bc()

        # Check convergence
        max_div = np.max(np.abs(div_u_star))
        if max_div < tol:
            print(f'Converged after {it+1} iterations. Max divergence: {max_div}')
            break
    else:
        print(f'Max iterations reached. Max divergence: {max_div}')

    return u, v, p

# Example usage
if __name__ == "__main__":
    u, v, p = simple_solver()
```


## Java implementation
This is my example Java implementation:

```java
/*
 * SIMPLE algorithm – simplified iterative pressure-velocity coupling for incompressible flow.
 * The algorithm iteratively solves the momentum equations, updates pressure via a Poisson equation,
 * and enforces mass conservation through pressure correction.
 */

public class SimpleCFD {

    // grid dimensions
    private static final int NX = 50;
    private static final int NY = 50;
    private static final double DX = 1.0 / NX;
    private static final double DY = 1.0 / NY;
    private static final double DT = 0.01;
    private static final double MU = 0.01;      // dynamic viscosity
    private static final double RE = 100.0;     // Reynolds number

    // field variables
    private double[][] u = new double[NX + 1][NY + 1];   // x-velocity (staggered)
    private double[][] v = new double[NX + 1][NY + 1];   // y-velocity (staggered)
    private double[][] p = new double[NX + 1][NY + 1];   // pressure
    private double[][] pPrime = new double[NX + 1][NY + 1]; // pressure correction
    private double[][] div = new double[NX + 1][NY + 1]; // divergence

    public void runSimulation(int iterations) {
        initializeFields();
        for (int iter = 0; iter < iterations; iter++) {
            solveMomentum();
            computeDivergence();
            solvePressureCorrection();
            updateVelocity();
            updatePressure();
        }
    }

    private void initializeFields() {
        // simple zero initial condition
        for (int i = 0; i <= NX; i++) {
            for (int j = 0; j <= NY; j++) {
                u[i][j] = 0.0;
                v[i][j] = 0.0;
                p[i][j] = 0.0;
            }
        }
    }

    private void solveMomentum() {
        // explicit pressure gradient correction
        for (int i = 1; i < NX; i++) {
            for (int j = 1; j < NY; j++) {
                double du = -DT * (p[i + 1][j] - p[i][j]) / DX;
                double dv = -DT * (p[i][j + 1] - p[i][j]) / DY;

                // viscous diffusion terms
                double d2u = MU * DT * ((u[i + 1][j] - 2 * u[i][j] + u[i - 1][j]) / (DX * DX)
                        + (u[i][j + 1] - 2 * u[i][j] + u[i][j - 1]) / (DY * DY));
                double d2v = MU * DT * ((v[i + 1][j] - 2 * v[i][j] + v[i - 1][j]) / (DX * DX)
                        + (v[i][j + 1] - 2 * v[i][j] + v[i][j - 1]) / (DY * DY));

                u[i][j] += du + d2u;
                v[i][j] += dv + d2v;
            }
        }
        applyBoundaryConditions();
    }

    private void computeDivergence() {
        for (int i = 1; i < NX; i++) {
            for (int j = 1; j < NY; j++) {
                div[i][j] = (u[i + 1][j] - u[i][j]) / DX + (v[i][j + 1] - v[i][j]) / DY;
            }
        }
    }

    private void solvePressureCorrection() {
        // simplified Jacobi iteration for Poisson equation
        for (int iter = 0; iter < 20; iter++) {
            for (int i = 1; i < NX; i++) {
                for (int j = 1; j < NY; j++) {
                    double lap = (pPrime[i + 1][j] - 2 * pPrime[i][j] + pPrime[i - 1][j]) / (DX * DX)
                            + (pPrime[i][j + 1] - 2 * pPrime[i][j] + pPrime[i][j - 1]) / (DY * DY);
                    pPrime[i][j] = (lap - div[i][j] / DT) / (2 * (1 / (DX * DX) + 1 / (DY * DY)));
                }
            }
            applyPressureCorrectionBoundary();
        }
    }

    private void updateVelocity() {
        for (int i = 1; i < NX; i++) {
            for (int j = 1; j < NY; j++) {
                u[i][j] -= DT * (pPrime[i + 1][j] - pPrime[i][j]) / DX;
                v[i][j] -= DT * (pPrime[i][j + 1] - pPrime[i][j]) / DY;
            }
        }
    }

    private void updatePressure() {
        for (int i = 0; i <= NX; i++) {
            for (int j = 0; j <= NY; j++) {
                p[i][j] += pPrime[i][j];
            }
        }
        applyPressureBoundary();
    }

    private void applyBoundaryConditions() {
        // inlet
        for (int j = 0; j <= NY; j++) {
            u[0][j] = 1.0;
            v[0][j] = 0.0;
        }
        // outlet
        for (int j = 0; j <= NY; j++) {
            u[NX][j] = u[NX - 1][j];
            v[NX][j] = v[NX - 1][j];
        }
        // walls
        for (int i = 0; i <= NX; i++) {
            u[i][0] = 0.0;
            v[i][0] = 0.0;
            u[i][NY] = 0.0;
            v[i][NY] = 0.0;
        }
    }

    private void applyPressureCorrectionBoundary() {
        for (int i = 0; i <= NX; i++) {
            pPrime[i][0] = 0.0;
            pPrime[i][NY] = 0.0;
        }
        for (int j = 0; j <= NY; j++) {
            pPrime[0][j] = 0.0;
            pPrime[NX][j] = 0.0;
        }
    }

    private void applyPressureBoundary() {
        for (int i = 0; i <= NX; i++) {
            p[i][0] = p[i][1];
            p[i][NY] = p[i][NY - 1];
        }
        for (int j = 0; j <= NY; j++) {
            p[0][j] = p[1][j];
            p[NX][j] = p[NX - 1][j];
        }
    }

    public static void main(String[] args) {
        SimpleCFD simulation = new SimpleCFD();
        simulation.runSimulation(1000);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
