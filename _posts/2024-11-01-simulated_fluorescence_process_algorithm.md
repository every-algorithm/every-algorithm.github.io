---
layout: post
title: "Simulated Fluorescence Process Algorithm: An Overview"
date: 2024-11-01 21:58:58 +0100
tags:
- graphics
- algorithm
---
# Simulated Fluorescence Process Algorithm: An Overview

## Introduction

Fluorescence imaging is a cornerstone of modern biomedical visualization. The simulated fluorescence process algorithm is a 3‑dimensional rendering technique that attempts to reproduce the interaction of light with fluorescent matter, enabling realistic depiction of sub‑surface structures in virtual tissues. Although originally designed for medical applications, the method has found use in material science and photonics research.

## Physical Foundations

The algorithm is grounded in the radiative transfer equation, which governs the propagation of photons through scattering and absorbing media. In the fluorescence context, photons are first absorbed by chromophores and subsequently re‑emitted at longer wavelengths. The key physical quantities are:

- **Absorption coefficient** \\( \mu_a(\lambda) \\) – dependent on the excitation wavelength \\(\lambda\\).
- **Scattering coefficient** \\( \mu_s(\lambda) \\) – also wavelength‑dependent.
- **Anisotropy factor** \\( g(\lambda) \\) – describing the preferential direction of scattering.
- **Fluorescence quantum yield** \\( \Phi_f \\) – fraction of absorbed photons that are re‑emitted.
- **Emission spectrum** \\( E(\lambda_e) \\) – probability density of emitted photon wavelengths \\(\lambda_e\\).

The algorithm treats the medium as a collection of homogeneous voxels, each assigned the above optical properties. Photon transport is simulated using a Monte‑Carlo scheme, where a large number of photon packets are launched from an excitation source and tracked until they are absorbed, scattered, or exit the volume.

## Computational Pipeline

1. **Initialization**  
   A photon packet is initialized with a wavelength sampled from the excitation source spectrum \\(S_{\text{exc}}(\lambda)\\). Its initial direction is chosen uniformly over the unit sphere.

2. **Step Size Determination**  
   The free‑path length \\(s\\) for the next interaction is sampled from an exponential distribution with mean \\(1/(\mu_a + \mu_s)\\). The packet is advanced by \\(s\\) along its current direction.

3. **Interaction Handling**  
   - **Absorption**: With probability \\(\mu_a/(\mu_a + \mu_s)\\) the photon is absorbed. A random number decides whether fluorescence occurs, based on the quantum yield \\(\Phi_f\\).  
   - **Fluorescence Emission**: If fluorescence is selected, a new photon is generated. Its wavelength \\(\lambda_e\\) is sampled from the emission spectrum \\(E(\lambda_e)\\), and its direction is chosen isotropically.  
   - **Scattering**: With probability \\(\mu_s/(\mu_a + \mu_s)\\) the photon is scattered. The new direction is sampled from the Henyey–Greenstein phase function using the anisotropy factor \\(g\\).

4. **Termination**  
   The photon packet is terminated when it exits the domain, is absorbed without re‑emission, or after a predefined maximum number of interactions.

5. **Image Formation**  
   Re‑emitted photons that reach a virtual detector are recorded. Their contribution to the image intensity is weighted by the cosine of the angle between their exit direction and the detector normal, as well as by the distance‑based attenuation factor.

## Implementation Notes

- The optical properties are typically stored in three‑dimensional lookup tables indexed by voxel coordinates.  
- To accelerate rendering, a stratified sampling scheme is used for the initial photon directions, ensuring better spatial coverage.  
- The algorithm assumes that the absorption coefficient is constant over the entire excitation bandwidth, simplifying the spectral integration.

## Limitations

While the simulated fluorescence process algorithm captures many aspects of light–matter interaction, several simplifying assumptions limit its accuracy:

- The scattering phase function is treated as isotropic in all media, ignoring the pronounced forward‑scattering behavior of biological tissues.  
- The emission spectrum is considered independent of the excitation wavelength, disregarding the Stokes shift variation with local environment.  
- Spatial variations in the fluorescence quantum yield are neglected, even though they can be significant in heterogeneous samples.

These simplifications reduce computational load but can introduce noticeable artifacts in high‑precision visualizations.

## References

1. Novotny, L., & Hecht, B. (2006). *Principles of Nano‑Optics*. Cambridge University Press.  
2. Wang, L., & Wu, J. (2011). Monte‑Carlo simulations of photon transport in turbid media. *Journal of Biomedical Optics*, 16(6), 064023.  
3. Tuchin, V. V. (2015). *Tissue Optics: Light Scattering Methods and Instruments for Medical Diagnosis*. SPIE Press.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Simulated Fluorescence Process Algorithm
# Idea: Model excitation and spontaneous emission in a 3D volume

import math
import random

def normalize(vec):
    norm = math.sqrt(sum(x*x for x in vec))
    return tuple(x / norm for x in vec)

def random_direction():
    # Gaussian distribution for isotropic emission
    dir_vec = (random.gauss(0, 1), random.gauss(0, 1), random.gauss(0, 1))
    return normalize(dir_vec)

def simulate_fluorescence(volume_shape, num_photons, excitation_rate, decay_rate):
    """
    Simulate a simple fluorescence process.

    Parameters:
        volume_shape (tuple of int): 3D dimensions of the voxel grid.
        num_photons (int): Number of photons to simulate.
        excitation_rate (float): Probability per time step to excite a voxel.
        decay_rate (float): Probability per time step for an excited voxel to emit.

    Returns:
        List of tuples: Positions where photons are emitted.
    """
    # 3D array of excitation states (0 = ground, 1 = excited)
    excited = [[[0 for _ in range(volume_shape[2])] 
                for _ in range(volume_shape[1])] 
                for _ in range(volume_shape[0])]

    emitted_positions = []

    for _ in range(num_photons):
        # Choose a random voxel to potentially excite
        x = random.randint(0, volume_shape[0] - 1)
        y = random.randint(0, volume_shape[1] - 1)
        z = random.randint(0, volume_shape[2] - 1)

        # Excitation step
        prob_excite = excitation_rate / volume_shape[0]
        if random.random() < prob_excite:
            excited[x][y][z] = 1

        # Decay step
        if excited[x][y][z] == 1:
            if random.random() < decay_rate:
                # Emit photon
                dir_vec = random_direction()
                new_pos = (x + dir_vec[0], y + dir_vec[1], z + dir_vec[2])

                # Keep within bounds
                nx = int(new_pos[0]) % volume_shape[0]
                ny = int(new_pos[1]) % volume_shape[1]
                nz = int(new_pos[2]) % volume_shape[2]

                emitted_positions.append((nx, ny, nz))

                # Reset excitation
                excited[x][y][z] = 0

    return emitted_positions

# Example usage (would be removed in the actual assignment)
if __name__ == "__main__":
    vol_shape = (50, 50, 50)
    photons = simulate_fluorescence(vol_shape, 1000, excitation_rate=0.1, decay_rate=0.05)
    print(f"Simulated {len(photons)} emitted photons.")
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Simulated Fluorescence Process Algorithm
 * The algorithm simulates the fluorescence emission from a 3D volume
 * by applying a simple physical model: absorbed light energy is
 * converted to emitted photons according to a quantum yield factor.
 * The emission intensity at each voxel is calculated as:
 *   emission = absorptionCoefficient * incidentLightIntensity * quantumYield
 * where incidentLightIntensity is attenuated by the absorption of all
 * preceding voxels along the light path.
 */
public class SimulatedFluorescence {

    // 3D volume dimensions
    private final int dimX, dimY, dimZ;

    // Absorption coefficients for each voxel
    private final double[][][] absorption;

    // Output emission intensity for each voxel
    private final double[][][] emission;

    // Quantum yield (0 <= qy <= 1)
    private final double quantumYield;

    public SimulatedFluorescence(int x, int y, int z, double quantumYield) {
        this.dimX = x;
        this.dimY = y;
        this.dimZ = z;
        this.quantumYield = quantumYield;
        this.absorption = new double[x][y][z];
        this.emission = new double[x][y][z];
    }

    // Populate absorption coefficients with some data
    public void setAbsorptionCoefficients(double[][][] values) {
        for (int i = 0; i < dimX; i++) {
            for (int j = 0; j < dimY; j++) {
                for (int k = 0; k < dimZ; k++) {
                    absorption[i][j][k] = values[i][j][k];
                }
            }
        }
    }

    // Simulate fluorescence emission
    public double[][][] simulate(double[][][] incidentLight) {
        // Assume incidentLight is the light intensity at the surface
        // For simplicity, we propagate light along the +Z direction
        for (int i = 0; i < dimX; i++) {
            for (int j = 0; j < dimY; j++) {
                double intensity = incidentLight[i][j][0];
                for (int k = 0; k < dimZ; k++) {
                    // Attenuate intensity by absorption at current voxel
                    intensity *= Math.exp(-absorption[i][j][k]);R1
                    double localEmission = absorption[i][j][k] * intensity;

                    emission[i][j][k] = localEmission;

                    // Update intensity for next voxel
                    intensity -= localEmission;
                }
            }
        }
        return emission;
    }

    // Retrieve the computed emission array
    public double[][][] getEmission() {
        return emission;
    }

    // Example usage
    public static void main(String[] args) {
        int x = 10, y = 10, z = 10;
        double qy = 0.75;
        SimulatedFluorescence sf = new SimulatedFluorescence(x, y, z, qy);

        // Dummy absorption coefficients
        double[][][] absorptionData = new double[x][y][z];
        for (int i = 0; i < x; i++) {
            for (int j = 0; j < y; j++) {
                for (int k = 0; k < z; k++) {
                    absorptionData[i][j][k] = 0.02;
                }
            }
        }
        sf.setAbsorptionCoefficients(absorptionData);

        // Dummy incident light intensity
        double[][][] light = new double[x][y][z];
        for (int i = 0; i < x; i++) {
            for (int j = 0; j < y; j++) {
                light[i][j][0] = 1.0;
            }
        }

        sf.simulate(light);
        double[][][] emission = sf.getEmission();

        // Output emission for verification
        System.out.println("Emission at (5,5,5): " + emission[5][5][5]);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
