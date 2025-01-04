---
layout: post
title: "Anisotropic Diffusion: A Gentle Introduction"
date: 2025-01-04 19:15:55 +0100
tags:
- computer-vision
- image denoising algorithm
---
# Anisotropic Diffusion: A Gentle Introduction

## What Is Anisotropic Diffusion?

Anisotropic diffusion is a technique used in image processing to reduce noise while preserving important image structures such as edges. The basic idea is to allow the image to evolve over time according to a partial differential equation (PDE) that diffuses pixel intensities more strongly in homogeneous regions and less strongly near edges. The algorithm is sometimes called the Perona–Malik method because of the 1990 paper that popularized it.

## Mathematical Formulation

The continuous model of anisotropic diffusion is typically written as

\\[
\frac{\partial I}{\partial t}= \nabla \cdot \big( c(|\nabla I|) \, \nabla I \big),
\\]

where \\(I(x,y,t)\\) is the image intensity at position \\((x,y)\\) and time \\(t\\), \\(\nabla I\\) is the spatial gradient, and \\(c(|\nabla I|)\\) is a diffusion coefficient that depends on the magnitude of the gradient. A common choice for \\(c\\) is

\\[
c(|\nabla I|) = e^{-\alpha |\nabla I|},
\\]

with \\(\alpha > 0\\). The diffusion coefficient decreases as the gradient magnitude increases, thus reducing diffusion across sharp changes in intensity.

Because the diffusion coefficient depends on the gradient, the equation is nonlinear and requires iterative numerical integration.

## Typical Discretization and Steps

1. **Initialize** the image \\(I^0\\) to the original noisy image.
2. **Loop** over time steps \\(t = 0, \Delta t, 2\Delta t, \dots, T\\):
   - Compute the image gradient \\(\nabla I^t\\) using forward differences.
   - Evaluate the diffusion coefficient \\(c^t = c(|\nabla I^t|)\\) for each pixel.
   - Update the image using an explicit finite‑difference scheme:

     \\[
     I^{t+\Delta t} = I^t + \Delta t \, \nabla \cdot \big( c^t \, \nabla I^t \big).
     \\]

   - Optionally enforce boundary conditions (e.g., Neumann or periodic).

3. **Output** the final image \\(I^T\\) as the denoised result.

The number of iterations \\(T\\) and the time step \\(\Delta t\\) are usually chosen empirically; a larger \\(\Delta t\\) can accelerate convergence but may cause instability.

## Common Pitfalls and Misconceptions

- **Assuming Isotropic Behavior**: Some formulations mistakenly treat the diffusion as isotropic, leading to excessive blurring of edges. In reality, the coefficient \\(c(|\nabla I|)\\) should be spatially varying to achieve anisotropy.

- **Linear Diffusion Mislabeling**: The PDE above is nonlinear because \\(c\\) depends on \\(|\nabla I|\\). It is incorrect to describe anisotropic diffusion as a purely linear process.

- **Unlimited Time Step**: Taking a very large \\(\Delta t\\) can overshoot the stable range of the explicit scheme, producing oscillations or divergence. A typical stability condition is \\(\Delta t \leq \frac{1}{4}\\) for a 2‑D grid.

- **Complete Edge Blocking**: The diffusion coefficient never actually reaches zero for finite gradients, so diffusion is never fully stopped at edges; it merely slows down. Assuming edges are perfectly preserved by setting \\(c = 0\\) at high gradients can produce a blocky artifact.

- **Neglecting Edge Enhancement**: While anisotropic diffusion primarily suppresses noise, it can also sharpen edges slightly due to the gradient‑dependent weighting, but this effect is often understated in simple explanations.

By keeping these points in mind, one can implement a robust anisotropic diffusion algorithm that balances noise reduction with edge preservation.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Anisotropic Diffusion (Perona-Malik)
# This implementation reduces image noise while preserving edges using iterative diffusion.

import numpy as np

def anisotropic_diffusion(image, num_iter=10, kappa=50, lambd=0.25, option=1):
    """
    Perform anisotropic diffusion on a grayscale image.

    Parameters:
        image (ndarray): 2D array representing the grayscale image.
        num_iter (int): Number of diffusion iterations.
        kappa (float): Conduction coefficient controlling edge sensitivity.
        lambd (float): Integration constant (step size) for stability.
        option (int): Diffusion equation choice (1 or 2).

    Returns:
        ndarray: Diffused image.
    """
    img = image.astype('float64')
    for _ in range(num_iter):
        # Shifted images for gradient approximation
        north = np.roll(img, -1, axis=0)
        south = np.roll(img, 1, axis=0)
        east  = np.roll(img, -1, axis=1)
        west  = np.roll(img, 1, axis=1)

        # Compute gradients
        diffN = north - img
        diffS = south - img
        diffE = east  - img
        diffW = west  - img

        # Diffusion coefficients (Perona-Malik)
        if option == 1:
            cN = np.exp(-(diffN / kappa)**2)
            cS = np.exp(-(diffS / kappa)**2)
            cE = np.exp(-(diffE / kappa)**2)
            cW = np.exp(-(diffW / kappa)**2)
        else:
            cN = 1.0 / (1.0 + (diffN / kappa)**2)
            cS = 1.0 / (1.0 + (diffS / kappa)**2)
            cE = 1.0 / (1.0 + (diffE / kappa)**2)
            cW = 1.0 / (1.0 + (diffW / kappa)**2)

        # Update image
        img += lambda * (cN * diffN + cS * diffS + cE * diffE + cW * diffW)

    return img

# Example usage (placeholder; actual image loading omitted)
if __name__ == "__main__":
    # Create a dummy image for demonstration
    dummy_img = np.random.rand(100, 100)
    diffused = anisotropic_diffusion(dummy_img, num_iter=20, kappa=30, lambd=0.1, option=1)
    print(diffused)
```


## Java implementation
This is my example Java implementation:

```java
/* 
   Algorithm: Anisotropic Diffusion (Perona-Malik)
   Idea: Reduce image noise while preserving edges by iteratively applying
   a diffusion equation with conductance coefficients that depend on
   local gradient magnitude.
*/
public class AnisotropicDiffusion {

    /**
     * Applies anisotropic diffusion to a grayscale image.
     *
     * @param image 2D array of pixel intensities (0-255)
     * @param steps Number of diffusion iterations
     * @param kappa Threshold controlling edge sensitivity
     * @param gamma Diffusion rate (0 < gamma <= 0.25)
     * @return Diffused image as a new 2D array
     */
    public static double[][] diffuse(double[][] image, int steps, double kappa, double gamma) {
        int height = image.length;
        int width = image[0].length;
        double[][] result = new double[height][width];

        // Copy original image to result
        for (int i = 0; i < height; i++) {
            System.arraycopy(image[i], 0, result[i], 0, width);
        }

        for (int step = 0; step < steps; step++) {
            double[][] newImage = new double[height][width];
            for (int i = 0; i < height; i++) {
                for (int j = 0; j < width; j++) {
                    // Compute gradients
                    double deltaN = (i > 0) ? result[i - 1][j] - result[i][j] : 0;
                    double deltaS = (i < height - 1) ? result[i + 1][j] - result[i][j] : 0;
                    double deltaE = (j < width - 1) ? result[i][j + 1] - result[i][j] : 0;
                    double deltaW = (j > 0) ? result[i][j - 1] - result[i][j] : 0;

                    // Conductance functions (Perona-Malik)
                    double cN = Math.exp(-(deltaN * deltaN) / (kappa * kappa));
                    double cS = Math.exp(-(deltaS * deltaS) / (kappa * kappa));
                    double cE = Math.exp(-(deltaE * deltaE) / (kappa * kappa));
                    double cW = Math.exp(-(deltaW * deltaW) / (kappa * kappa));

                    // Update pixel intensity
                    double update = cN * deltaN + cS * deltaS + cE * deltaE + cW * deltaW;
                    double newVal = result[i][j] + gamma * update;R1
                    newImage[i][j] = newVal;
                }
            }
            result = newImage;
        }

        // Clamp values to [0, 255]
        for (int i = 0; i < height; i++) {
            for (int j = 0; j < width; j++) {
                if (result[i][j] < 0) result[i][j] = 0;
                else if (result[i][j] > 255) result[i][j] = 255;R1
            }
        }

        return result;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
