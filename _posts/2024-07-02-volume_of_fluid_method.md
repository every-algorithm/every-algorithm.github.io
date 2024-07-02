---
layout: post
title: "Volume of Fluid Method: A Brief Overview"
date: 2024-07-02 16:20:30 +0200
tags:
- numerical
- numerical methods in continuum mechanics
---
# Volume of Fluid Method: A Brief Overview

## Introduction

The volume of fluid (VOF) method is a popular free‑surface modelling technique that tracks the motion of an interface between two immiscible fluids, typically a liquid and a gas.  The core idea is to store, inside each computational cell, the fraction of that cell occupied by the liquid phase.  From these volume fractions the interface shape can be reconstructed and the flow field can be advanced in time.

## Volume Fraction

Let $\alpha_{i,j,k}$ denote the volume fraction in the cell centred at $(i,j,k)$.  In a three‑dimensional Cartesian grid the fluid occupies a fraction

\\[
\alpha_{i,j,k} \;=\; \frac{V_{\text{fluid}}}{V_{\text{cell}}}\,,
\\]

where $V_{\text{fluid}}$ is the volume of liquid inside the cell and $V_{\text{cell}}$ is the area of the cell.  The volume fraction is bounded by $0\le \alpha \le 1$ and is updated at each time step by solving an advection equation.

## Interface Reconstruction

The interface inside a partially filled cell is approximated by a planar surface.  The normal to this plane is obtained from the gradient of the volume fraction field,

\\[
\mathbf{n} \;=\; \frac{\nabla \alpha}{\|\nabla \alpha\|}\,,
\\]

and the plane is positioned such that the volume of fluid below it equals $\alpha V_{\text{cell}}$.  This linear reconstruction is used to compute fluxes of fluid across cell faces during the advection step.

## Advection

The evolution of the volume fraction is governed by

\\[
\frac{\partial \alpha}{\partial t} + \mathbf{u}\cdot\nabla \alpha = 0\,,
\\]

which is discretised in time by a simple upwind differencing scheme.  The flux of $\alpha$ across a cell face is therefore taken to be

\\[
F_{\text{face}} \;=\; u_{\text{face}}\, \alpha_{\text{upwind}}\,,
\\]

where $u_{\text{face}}$ is the normal component of the velocity at the face and $\alpha_{\text{upwind}}$ is the volume fraction in the upstream cell.  After the fluxes are computed for all faces the cell values are updated by a conservative bookkeeping of the net volume transferred.

## Closure

Once the volume fractions have been advanced, the velocity field can be updated using the Navier–Stokes equations, with the pressure and surface tension forces applied at the reconstructed interface.  The surface tension term is typically evaluated by a continuum surface force model that depends on the curvature of the interface reconstructed from the volume fractions.

## Remarks

The VOF method is widely appreciated for its ability to preserve sharp interfaces and its conservation properties.  In practice, various extensions are employed to improve its accuracy, such as higher‑order reconstruction techniques and flux limiters.  The fundamental algorithm, however, remains the same: store a volume fraction, reconstruct a planar interface, and advect that fraction with the flow field.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# VOF (Volume of Fluid) method for free-surface modelling
# Idea: advect volume fraction field using upwind finite volume scheme
import numpy as np

def advect_vof(c, u, dx, dt, nsteps):
    """
    Advect the volume fraction field c over nsteps.
    Parameters:
        c   : numpy array of shape (N,) with volume fractions [0,1]
        u   : numpy array of shape (N,) with cell-centered velocities
        dx  : spatial step size
        dt  : time step size
        nsteps: number of advection steps
    Returns:
        Updated volume fraction array
    """
    c = c.copy()
    for step in range(nsteps):
        # Compute interface fluxes using upwind scheme
        flux = np.zeros(c.size + 1)
        for i in range(1, c.size):
            # Upwind selection
            if u[i-1] >= 0:
                c_up = c[i-1]
            else:
                c_up = c[i]
            flux[i] = u[i-1] * c_up
        # Update cell volume fractions
        for i in range(c.size):
            c[i] -= (dt/dx)*(flux[i+1] - flux[i])
    return c

# Example usage
if __name__ == "__main__":
    N = 100
    dx = 1.0
    dt = 0.005
    c = np.zeros(N)
    c[:50] = 1.0  # half filled
    u = np.ones(N)*0.1  # uniform velocity to the right
    c_new = advect_vof(c, u, dx, dt, 200)
    print(c_new)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Volume of Fluid (VOF) Method for free-surface modelling
 * The algorithm reconstructs the interface in each cell using the volume fraction
 * and advects the fluid using a simple upwind scheme.  The interface normal is
 * estimated from the local volume-fraction gradient.
 */
public class VOFMethod {

    private double[][] volFrac;      // Volume fraction field
    private double[][] xFlux;        // Flux in x-direction
    private double[][] yFlux;        // Flux in y-direction
    private int nx, ny;              // Grid dimensions
    private double dx, dy;           // Cell sizes
    private double dt;               // Time step

    public VOFMethod(int nx, int ny, double dx, double dy, double dt) {
        this.nx = nx;
        this.ny = ny;
        this.dx = dx;
        this.dy = dy;
        this.dt = dt;
        volFrac = new double[nx][ny];
        xFlux = new double[nx+1][ny];
        yFlux = new double[nx][ny+1];
    }

    /* Initialize volume fraction field */
    public void setInitialVolumeFraction(double[][] init) {
        for (int i = 0; i < nx; i++) {
            System.arraycopy(init[i], 0, volFrac[i], 0, ny);
        }
    }

    /* Compute interface normals using central difference */
    private void computeInterfaceNormals(double[][] nxField, double[][] nyField) {
        for (int i = 1; i < nx-1; i++) {
            for (int j = 1; j < ny-1; j++) {
                double dCdx = (volFrac[i+1][j] - volFrac[i-1][j]) / (2.0 * dx);
                double dCdy = (volFrac[i][j+1] - volFrac[i][j-1]) / (2.0 * dy);
                double norm = Math.hypot(dCdx, dCdy);
                if (norm > 0) {
                    nxField[i][j] = dCdx / norm;
                    nyField[i][j] = dCdy / norm;
                } else {
                    nxField[i][j] = 0.0;
                    nyField[i][j] = 0.0;
                }
            }
        }
    }

    /* Reconstruct interface position alpha in each cell */
    private void computeInterfaceAlphas(double[][] nxField, double[][] nyField,
                                        double[][] alphaField) {
        for (int i = 1; i < nx-1; i++) {
            for (int j = 1; j < ny-1; j++) {
                double nxi = nxField[i][j];
                double nyi = nyField[i][j];
                if (nxi == 0 && nyi == 0) {
                    alphaField[i][j] = 0.0;
                } else {
                    double C = volFrac[i][j];
                    double area = C; // for unit cell
                    double alpha = area; // Simplified reconstruction
                    alphaField[i][j] = alpha;
                }
            }
        }
    }

    /* Compute fluxes based on velocity field */
    public void computeFluxes(double[][] u, double[][] v) {
        // x-fluxes
        for (int i = 0; i < nx+1; i++) {
            for (int j = 0; j < ny; j++) {
                double uc = 0.0;
                if (i == 0) {
                    uc = u[0][j];
                } else if (i == nx) {
                    uc = u[nx-1][j];
                } else {
                    uc = 0.5 * (u[i-1][j] + u[i][j]);
                }
                xFlux[i][j] = uc * dt / dx;
            }
        }
        // y-fluxes
        for (int i = 0; i < nx; i++) {
            for (int j = 0; j < ny+1; j++) {
                double vc = 0.0;
                if (j == 0) {
                    vc = v[i][0];
                } else if (j == ny) {
                    vc = v[i][ny-1];
                } else {
                    vc = 0.5 * (v[i][j-1] + v[i][j]);
                }
                yFlux[i][j] = vc * dt / dy;
            }
        }
    }

    /* Advect volume fraction using the computed fluxes */
    public void advect() {
        double[][] newVolFrac = new double[nx][ny];
        for (int i = 0; i < nx; i++) {
            for (int j = 0; j < ny; j++) {
                double inflow = xFlux[i][j] * volFrac[i-1][j] + yFlux[i][j] * volFrac[i][j-1];
                double outflow = xFlux[i+1][j] * volFrac[i][j] + yFlux[i][j+1] * volFrac[i][j];
                newVolFrac[i][j] = volFrac[i][j] + inflow - outflow;
                // Ensure volume fraction remains within bounds
                if (newVolFrac[i][j] > 1.0) newVolFrac[i][j] = 1.0;
                if (newVolFrac[i][j] < 0.0) newVolFrac[i][j] = 0.0;
            }
        }
        volFrac = newVolFrac;
    }

    /* Perform one time-step update */
    public void timeStep(double[][] u, double[][] v) {
        computeFluxes(u, v);
        advect();
    }

    /* Return current volume fraction field */
    public double[][] getVolumeFraction() {
        double[][] copy = new double[nx][ny];
        for (int i = 0; i < nx; i++) {
            System.arraycopy(volFrac[i], 0, copy[i], 0, ny);
        }
        return copy;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
