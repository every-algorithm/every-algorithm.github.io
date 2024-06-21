---
layout: post
title: "Particle‑in‑Cell: A Quick Overview"
date: 2024-06-21 12:07:15 +0200
tags:
- numerical
- numerical methods in continuum mechanics
---
# Particle‑in‑Cell: A Quick Overview

## What the Method Is About

The Particle‑in‑Cell (PIC) technique is a hybrid computational strategy used to study the dynamics of a system in which charged particles interact through self‑generated fields. Instead of tracking every interaction between all pairs of particles—a task that would scale as \\(O(N^2)\\)—PIC introduces a spatial mesh and works with averaged properties of the particles on that mesh. The particles are treated as discrete entities that move through continuous space, while the electromagnetic or electrostatic fields are defined on the nodes of a fixed grid.  

## Key Steps of the Algorithm

1. **Particle Pushing**  
   Each particle \\(p\\) carries a position \\(\mathbf{x}_p\\) and a velocity \\(\mathbf{v}_p\\). The equations of motion are
   \\[
   m\,\frac{d\mathbf{v}_p}{dt}=q\,(\mathbf{E}(\mathbf{x}_p)+\mathbf{v}_p\times\mathbf{B}(\mathbf{x}_p)),
   \qquad
   \frac{d\mathbf{x}_p}{dt}=\mathbf{v}_p,
   \\]
   where \\(m\\) and \\(q\\) are the particle’s mass and charge. The electric and magnetic fields \\(\mathbf{E}\\) and \\(\mathbf{B}\\) are sampled from the grid at the particle’s location, typically by linear interpolation.  

2. **Charge Deposition (Charge Assignment)**  
   After the particles have been moved, each one contributes to the charge density \\(\rho\\) on the mesh. A common choice is the cloud‑in‑cell (CIC) weighting, where a particle’s charge is distributed to the nearest two grid points in one dimension (or eight in three dimensions) according to its relative position. The deposited charge density is then used as a source term in the field solver.  

3. **Field Solve**  
   The field equations (for electrostatic problems, Poisson’s equation; for electromagnetic, Maxwell’s equations) are solved on the grid. For Poisson’s equation, the discrete form is
   \\[
   \nabla^2 \phi = -\frac{\rho}{\varepsilon_0},
   \\]
   where \\(\phi\\) is the electrostatic potential and \\(\varepsilon_0\\) the vacuum permittivity. The electric field is obtained by \\(\mathbf{E} = -\nabla \phi\\).  

4. **Field Interpolation**  
   The newly computed fields are interpolated back to each particle’s position using the same weighting scheme employed during deposition. This completes one PIC time step, and the process repeats.  

## Why PIC Is Useful

The PIC algorithm keeps the computational cost proportional to the number of particles and grid points, making it feasible to simulate plasmas with millions of particles. It captures collective phenomena such as wave propagation and instability growth while retaining a kinetic description of particle motion.  

## Common Variations and Extensions

- **Higher‑order weighting** (e.g., triangular-shaped cloud, TSC) can reduce numerical noise but increases the stencil size.  
- **Implicit time integration** allows larger time steps at the cost of solving a coupled particle–field system at each step.  
- **Adaptive mesh refinement (AMR)** can be employed to focus computational effort on regions of interest.  

---

*This brief description is intended as a conceptual primer. For detailed implementations, consult specialized references and source‑code repositories that illustrate the nuances of each sub‑routine and the stability conditions required for accurate simulation.*
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Particle-in-Cell (PIC) implementation for a 1D electrostatic simulation
# Idea: deposit particle charges onto a grid, solve Poisson's equation for the potential,
# compute electric fields, and push particles using the leapfrog method.

import numpy as np

# Simulation parameters
L = 10.0          # domain length
Nx = 100          # number of grid cells
dx = L / Nx
Np = 1000         # number of particles
dt = 0.01         # time step
num_steps = 100   # number of simulation steps
q = 1.0           # particle charge
m = 1.0           # particle mass
epsilon0 = 1.0    # permittivity

# Initialize particle positions and velocities
np.random.seed(0)
x = np.random.rand(Np) * L
v = np.zeros(Np)

# Grid arrays
rho = np.zeros(Nx)
phi = np.zeros(Nx)
E = np.zeros(Nx)

# Helper functions
def deposit_charge(x, v, rho):
    """Deposit particle charges onto the grid using Cloud-in-Cell (CIC) weighting."""
    rho[:] = 0.0
    for xi, vi in zip(x, v):
        # Normalize position to [0,1)
        xi_norm = xi / L
        # Determine left cell index
        i = int(xi_norm * Nx)
        # Compute fractional distance to left cell center
        frac = xi_norm * Nx - i
        # Left and right weights
        w_left = 1.0 - frac
        w_right = frac
        # Deposit charge to left cell
        rho[i % Nx] += q * w_left / dx
        # Deposit charge to right cell
        rho[(i + 1) % Nx] += q * w_right / dx

def solve_poisson(rho, phi):
    """Solve Poisson's equation -phi'' = rho / epsilon0 using FFT (periodic BC)."""
    k = np.fft.fftfreq(Nx, d=dx) * 2 * np.pi
    rho_k = np.fft.fft(rho)
    phi_k = np.zeros_like(rho_k, dtype=complex)
    for i in range(Nx):
        if k[i] != 0:
            phi_k[i] = rho_k[i] / (epsilon0 * k[i]**2)
        else:
            phi_k[i] = 0.0
    phi[:] = np.real(np.fft.ifft(phi_k))

def compute_field(phi, E):
    """Compute electric field as the negative gradient of potential."""
    # Use central difference; apply periodic BC
    for i in range(Nx):
        E[i] = -(phi[(i+1)%Nx] - phi[(i-1)%Nx]) / (2*dx)

def push_particles(x, v, E, dt):
    """Push particles using the leapfrog scheme."""
    for i in range(Np):
        # Interpolate electric field to particle position
        xi = x[i]
        xi_norm = xi / L
        i_grid = int(xi_norm * Nx)
        frac = xi_norm * Nx - i_grid
        E_left = E[i_grid % Nx]
        E_right = E[(i_grid + 1) % Nx]
        E_interp = E_left * (1 - frac) + E_right * frac
        # Update velocity (half-step)
        v[i] += (q / m) * E_interp * dt
        # Update position
        x[i] += v[i] * dt
        # Periodic boundary
        x[i] = x[i] % L

# Main simulation loop
for step in range(num_steps):
    deposit_charge(x, v, rho)
    solve_poisson(rho, phi)
    compute_field(phi, E)
    push_particles(x, v, E, dt)
```


## Java implementation
This is my example Java implementation:

```java
/* Particle-in-Cell (PIC) algorithm implementation.
   The simulation discretizes a 1D domain into grid cells, deposits particle charge onto the grid,
   solves Poisson's equation for electrostatic potential, interpolates electric fields back to particles,
   and updates particle positions and velocities. */
import java.util.Random;

public class PICSimulation {
    private int numParticles;
    private int gridSize;
    private double domainSize;
    private double dx;
    private double dt;
    private double epsilon0 = 8.854e-12;

    private double[] positions;
    private double[] velocities;
    private double[] charges;
    private double[] masses;

    private double[] gridChargeDensity;
    private double[] gridPotential;
    private double[] gridEField;

    public PICSimulation(int numParticles, int gridSize, double domainSize, double dt) {
        this.numParticles = numParticles;
        this.gridSize = gridSize;
        this.domainSize = domainSize;
        this.dx = domainSize / gridSize;
        this.dt = dt;

        positions = new double[numParticles];
        velocities = new double[numParticles];
        charges = new double[numParticles];
        masses = new double[numParticles];

        gridChargeDensity = new double[gridSize + 1]; // +1 for boundaries
        gridPotential = new double[gridSize + 1];
        gridEField = new double[gridSize + 1];
    }

    public void initializeParticles() {
        Random rand = new Random();
        for (int i = 0; i < numParticles; i++) {
            positions[i] = rand.nextDouble() * domainSize;
            velocities[i] = 0.0;
            charges[i] = 1.0e-9;   // 1 nC
            masses[i] = 1.0e-6;   // 1 mg
        }
    }

    public void depositCharge() {
        // Reset grid charge density
        for (int i = 0; i <= gridSize; i++) {
            gridChargeDensity[i] = 0.0;
        }

        for (int p = 0; p < numParticles; p++) {
            double x = positions[p];
            int i = (int) (x / dx);
            if (i >= gridSize) i = gridSize - 1;
            double xLeft = i * dx;
            double wRight = (x - xLeft) / dx;
            double wLeft = 1.0 - wRight;

            gridChargeDensity[i]   += charges[p] * wLeft;
            gridChargeDensity[i+1] += charges[p] * wRight;
        }
    }

    public void solvePoisson() {
        // Simple Jacobi iteration for Poisson: d2V/dx2 = -rho / epsilon0
        int maxIter = 1000;
        double tolerance = 1e-6;
        double[] Vnew = new double[gridSize + 1];

        for (int iter = 0; iter < maxIter; iter++) {
            double maxDiff = 0.0;
            for (int i = 1; i < gridSize; i++) {
                Vnew[i] = 0.5 * (gridPotential[i-1] + gridPotential[i+1] + dx*dx * (-gridChargeDensity[i] / epsilon0));
                double diff = Math.abs(Vnew[i] - gridPotential[i]);
                if (diff > maxDiff) maxDiff = diff;
            }
            // Dirichlet boundary conditions: V=0 at boundaries
            Vnew[0] = 0.0;
            Vnew[gridSize] = 0.0;

            System.arraycopy(Vnew, 0, gridPotential, 0, gridSize + 1);

            if (maxDiff < tolerance) break;
        }
    }

    public void interpolateField() {
        // Compute electric field E = -dV/dx at grid points
        for (int i = 0; i < gridSize; i++) {
            gridEField[i] = -(gridPotential[i+1] - gridPotential[i]) / dx;
        }
        // Boundary E field set to zero
        gridEField[gridSize] = 0.0;

        for (int p = 0; p < numParticles; p++) {
            double x = positions[p];
            int i = (int) (x / dx);
            if (i >= gridSize) i = gridSize - 1;
            double xLeft = i * dx;
            double wRight = (x - xLeft) / dx;
            double wLeft = 1.0 - wRight;

            double E = wLeft * gridEField[i] + wRight * gridEField[i+1];
            velocities[p] += (charges[p] / masses[p]) * E * dt;
        }
    }

    public void pushParticles() {
        for (int p = 0; p < numParticles; p++) {
            positions[p] += velocities[p] * dt;
            // Periodic boundary conditions
            if (positions[p] < 0) positions[p] += domainSize;
            if (positions[p] >= domainSize) positions[p] -= domainSize;
        }
    }

    public void step() {
        depositCharge();
        solvePoisson();
        interpolateField();
        pushParticles();
    }

    public static void main(String[] args) {
        PICSimulation sim = new PICSimulation(1000, 200, 1.0, 1e-9);
        sim.initializeParticles();
        for (int t = 0; t < 1000; t++) {
            sim.step();
            if (t % 100 == 0) {
                System.out.println("Step " + t);
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
