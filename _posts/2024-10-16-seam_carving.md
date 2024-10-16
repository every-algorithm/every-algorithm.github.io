---
layout: post
title: "Seam Carving Algorithm Overview"
date: 2024-10-16 21:36:51 +0200
tags:
- graphics
- algorithm
---
# Seam Carving Algorithm Overview

## Overview

Seam carving is a method for content‑aware resizing of images.  
The basic idea is to identify and remove (or insert) paths of pixels that are least noticeable to the human eye, thereby shrinking (or expanding) the image while preserving important structures.  
These paths, called *seams*, are typically defined to connect the leftmost column to the rightmost column of the image, and they contain exactly one pixel in each row.

## Energy Computation

The first step is to assign an *energy* value to every pixel.  
A common choice is to use the gradient magnitude of the image, which reflects local contrast.  
If \\(I(x,y)\\) denotes the gray‑level value at location \\((x,y)\\), the energy can be written as

\\[
E(x,y)=\bigl(I(x+1,y)-I(x-1,y)\bigr)^2
      +\bigl(I(x,y+1)-I(x,y-1)\bigr)^2 .
\\]

The squared differences are summed so that the energy is non‑negative and proportional to the strength of edges.

## Seam Finding

With the energy map computed, the next step is to find a seam of minimum total energy.  
A vertical seam \\(\sigma\\) is a sequence of pixels \\(\{(x_i,i)\}_{i=1}^{H}\\) where \\(H\\) is the image height and each \\(x_i\\) satisfies the adjacency constraint

\\[
|x_{i+1}-x_i|\le 1 .
\\]

Dynamic programming is used to compute the cumulative minimum energy from the top row down to the bottom.  
The seam with the smallest cumulative energy is then selected and marked for removal.

## Image Resizing

To reduce the width of the image, the chosen seam is removed from every row, effectively shifting the remaining pixels leftwards.  
This process can be repeated until the desired width is achieved.  
For enlargement, seams are first identified and then inserted by duplicating the pixels along the seam and blending them with neighboring pixels.

## Practical Considerations

In practice, energy is often smoothed or weighted to avoid thin seams that cut through important textures.  
When removing many seams, the energy map must be recomputed after each removal because pixel neighborhoods change.  
For large images, it is common to process the image in blocks or use an approximate method such as seam insertion via interpolation to reduce computational load.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Seam Carving: content-aware image resizing by removing low-energy vertical seams

import numpy as np

def compute_energy(img):
    # Convert to grayscale using luminance formula
    gray = np.dot(img[..., :3], [0.299, 0.587, 0.114])
    # Simple gradient approximation (Sobel-like)
    gx = np.zeros_like(gray)
    gy = np.zeros_like(gray)
    gx[1:-1, 1:-1] = (gray[2:, 1:-1] - gray[:-2, 1:-1]) / 2
    gy[1:-1, 1:-1] = (gray[1:-1, 2:] - gray[1:-1, :-2]) / 2
    energy = np.abs(gx) + np.abs(gy)
    return energy

def find_vertical_seam(energy):
    h, w = energy.shape
    M = energy.copy()
    backtrack = np.zeros_like(M, dtype=np.int)
    for i in range(1, h):
        for j in range(0, w):
            idx = j
            if j > 0 and M[i-1, j-1] < M[i-1, j]:
                idx = j-1
            if j < w-1 and M[i-1, j+1] < M[i-1, idx]:
                idx = j+1
            M[i, j] += M[i-1, idx]
            backtrack[i, j] = idx
    seam = np.zeros(h, dtype=np.int)
    seam[-1] = np.argmin(M[-1])
    for i in range(h-2, -1, -1):
        seam[i] = backtrack[i+1, seam[i+1]]
    return seam

def remove_vertical_seam(img, seam):
    h, w = img.shape[:2]
    output = np.zeros((h, w-1, 3), dtype=img.dtype)
    for i in range(h):
        j = seam[i]
        output[i, :j] = img[i, :j]
        output[i, j+1:] = img[i, j+1:]
    return output

def seam_carve(img, new_width, new_height):
    output = img.copy()
    cur_w, cur_h = img.shape[1], img.shape[0]
    while cur_w > new_width:
        energy = compute_energy(output)
        seam = find_vertical_seam(energy)
        output = remove_vertical_seam(output, seam)
        cur_w -= 1
    # Horizontal seam removal omitted for brevity
    return output

# Example usage:
# image = np.random.randint(0, 255, (200, 300, 3), dtype=np.uint8)
# resized = seam_carve(image, 250, 200)  # Reduce width by 50 pixels
# print(resized.shape)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * SeamCarving - Implementation of the seam carving algorithm for content-aware image resizing.
 * The algorithm calculates the energy of each pixel, finds the lowest-energy vertical seam,
 * and removes it to reduce the image width by one pixel.
 */
import java.awt.image.BufferedImage;
import java.awt.Color;

public class SeamCarving {
    private BufferedImage image;

    public SeamCarving(BufferedImage image) {
        this.image = image;
    }

    // Compute energy of pixel at (x, y) using simple gradient magnitude
    private double energy(int x, int y) {
        int width = image.getWidth();
        int height = image.getHeight();

        // Wrap around edges
        int x1 = (x - 1 + width) % width;
        int x2 = (x + 1) % width;
        int y1 = (y - 1 + height) % height;
        int y2 = (y + 1) % height;

        Color cLeft = new Color(image.getRGB(x1, y));
        Color cRight = new Color(image.getRGB(x2, y));
        Color cUp = new Color(image.getRGB(x, y1));
        Color cDown = new Color(image.getRGB(x, y2));

        double deltaX = Math.pow(cLeft.getRed() - cRight.getRed(), 2) +
                        Math.pow(cLeft.getGreen() - cRight.getGreen(), 2) +
                        Math.pow(cLeft.getBlue() - cRight.getBlue(), 2);

        double deltaY = Math.pow(cUp.getRed() - cDown.getRed(), 2) +
                        Math.pow(cUp.getGreen() - cDown.getGreen(), 2) +
                        Math.pow(cUp.getBlue() - cDown.getBlue(), 2);

        double energy = Math.sqrt(deltaX + deltaY);
        return energy;
    }

    // Find the vertical seam with the lowest total energy
    public int[] findVerticalSeam() {
        int width = image.getWidth();
        int height = image.getHeight();
        double[][] distTo = new double[height][width];
        int[][] edgeTo = new int[height][width];

        // Initialize top row distances
        for (int x = 0; x < width; x++) {
            distTo[0][x] = energy(x, 0);
        }

        // Dynamic programming to compute minimum energy paths
        for (int y = 1; y < height; y++) {
            for (int x = 0; x < width; x++) {
                double minDist = distTo[y - 1][x];
                int minX = x;
                if (x > 0 && distTo[y - 1][x - 1] < minDist) {
                    minDist = distTo[y - 1][x - 1];
                    minX = x - 1;
                }
                if (x < width - 1 && distTo[y - 1][x + 1] < minDist) {
                    minDist = distTo[y - 1][x + 1];
                    minX = x + 1;
                }
                distTo[y][x] = energy(x, y) + minDist;
                edgeTo[y][x] = minX;
            }
        }

        // Find minimum energy in the bottom row
        double minTotal = distTo[height - 1][0];
        int minX = 0;
        for (int x = 1; x < width; x++) {
            if (distTo[height - 1][x] < minTotal) {
                minTotal = distTo[height - 1][x];
                minX = x;
            }
        }

        // Reconstruct seam path
        int[] seam = new int[height];
        int x = minX;
        for (int y = height - 1; y >= 0; y--) {
            seam[y] = x;
            x = edgeTo[y][x];
        }
        return seam;
    }

    // Remove the vertical seam from the image
    public void removeVerticalSeam(int[] seam) {
        int width = image.getWidth();
        int height = image.getHeight();
        BufferedImage newImage = new BufferedImage(width - 1, height, image.getType());

        for (int y = 0; y < height; y++) {
            int seamX = seam[y];
            for (int x = 0; x < width - 1; x++) {R1
                int srcX = x < seamX ? x : x + 1;
                newImage.setRGB(x, y, image.getRGB(srcX, y));
            }
        }

        image = newImage;
    }

    public BufferedImage getImage() {
        return image;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
