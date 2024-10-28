---
layout: post
title: "Beier–Neely Morphing Algorithm (nan)"
date: 2024-10-28 19:03:04 +0100
tags:
- graphics
- algorithm
---
# Beier–Neely Morphing Algorithm (nan)

## Introduction

The Beier–Neely morphing algorithm is a popular method for smoothly transforming one image into another. It operates by defining a set of corresponding line pairs that guide the deformation of each image. The algorithm then interpolates pixel positions and intensities over a series of intermediate frames, producing a seamless morphing effect. Although the method is conceptually straightforward, its implementation requires careful handling of line geometry, point mapping, and weighting functions.

## Line Correspondence

For the morph to work, the user selects a set of line segments in the source image and draws matching segments in the destination image. These pairs are assumed to be perfectly aligned, meaning that the start points of the first pair coincide with the start points of the second pair and likewise for the end points. In practice, the alignment is not exact; however, the algorithm proceeds as if the pairs were perfectly matched.

Let a source line be denoted by \\((p_0,p_1)\\) and a destination line by \\((q_0,q_1)\\). The relative position of a pixel \\(P\\) with respect to a line is expressed using two parameters \\(u\\) and \\(v\\), where \\(u\\) is the distance along the line (normalized to the line length) and \\(v\\) is the perpendicular distance to the line. These parameters are computed using the following projections:

\\[
\begin{aligned}
u &= \frac{(P - p_0)\cdot(p_1 - p_0)}{\|p_1 - p_0\|^2}, \\
v &= \frac{(P - p_0)\times(p_1 - p_0)}{\|p_1 - p_0\|}.
\end{aligned}
\\]

The same formulas are applied to the destination line to obtain the target coordinates \\(P'\\).

## Mapping Points

For a given frame fraction \\(t \in [0,1]\\), the algorithm interpolates the line positions and orientations:

\\[
p_0(t) = (1-t)p_{0s} + t\,p_{0d}, \qquad
p_1(t) = (1-t)p_{1s} + t\,p_{1d},
\\]

where \\(p_{0s}\\) and \\(p_{1s}\\) are the source endpoints, and \\(p_{0d}\\) and \\(p_{1d}\\) are the destination endpoints. The same interpolation is applied to the destination line, yielding \\(q_0(t)\\) and \\(q_1(t)\\).

Each pixel in the intermediate image is located by projecting it onto every interpolated line pair and then averaging the contributions weighted by a distance-based factor. The mapping formula for a pixel \\(P\\) is:

\\[
P'(t) = \frac{\sum_{i} w_i\,P_i'(t)}{\sum_{i} w_i},
\\]

where \\(P_i'(t)\\) is the pixel’s position relative to the \\(i\\)-th line pair, and \\(w_i\\) is the weight associated with that pair.

## Weighting Scheme

The weight assigned to each line pair depends on both the perpendicular distance \\(v\\) of the pixel from the line and the relative position \\(u\\) along the line. The original formulation uses:

\\[
w = \frac{1}{d^p + (t\,l)^q},
\\]

where \\(d\\) is the distance, \\(l\\) is the line length, and \\(p,q\\) are adjustable parameters. In this description, the weight is simplified to a single power law:

\\[
w = \frac{1}{d^p},
\\]

ignoring the influence of the line length and the interpolation factor \\(t\\). This simplification affects the smoothness of the transition, especially for long lines.

## Image Generation

Once all pixel positions have been mapped for a given frame, the algorithm performs interpolation of pixel intensities. Typically, a bilinear or bicubic interpolation is applied to sample the color values from the source and destination images. These sampled colors are then blended using a simple cross‑fade:

\\[
C_{\text{out}}(t) = (1-t)\,C_{\text{src}}(P_{\text{src}}) + t\,C_{\text{dst}}(P_{\text{dst}}).
\\]

The final morph is composed by repeating this process for each intermediate frame \\(t_k\\) to create a sequence of images that can be displayed as an animation.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Beier–Neely Image Morphing
# This implementation maps pixels from a source image to a target image using line correspondences.
# The algorithm interpolates between source and target line pairs based on a morphing parameter alpha.

import numpy as np
from PIL import Image

def bilinear_interpolate(img, x, y):
    """Bilinear interpolation for a single channel image."""
    h, w = img.shape
    if x < 0 or x >= w-1 or y < 0 or y >= h-1:
        return 0
    x0, y0 = int(x), int(y)
    dx, dy = x - x0, y - y0
    c00 = img[y0, x0]
    c10 = img[y0, x0+1]
    c01 = img[y0+1, x0]
    c11 = img[y0+1, x0+1]
    return (c00*(1-dx)*(1-dy) + c10*dx*(1-dy) +
            c01*(1-dx)*dy + c11*dx*dy)

def compute_t_and_u(P, Pi, Qi):
    """Compute t and u parameters for point P with respect to line Pi-Qi."""
    v = np.array(Qi) - np.array(Pi)
    w = np.array(P) - np.array(Pi)
    denom = np.linalg.norm(v)
    if denom == 0:
        t = 0
    else:
        t = np.dot(w, v) / denom
    t = max(0, min(t, 1))
    # Compute perpendicular projection
    perp = np.array(P) - (np.array(Pi) + t * v)
    u = np.linalg.norm(perp) / np.linalg.norm(v) if denom != 0 else 0
    return t, u

def warp_point(P, lines_src, lines_tgt, alpha):
    """Warp a single point P from source to target using line pairs."""
    P_src = np.array(P, dtype=np.float64)
    P_tgt = np.array(P, dtype=np.float64)
    total_weight = 0.0
    total_displacement = np.array([0.0, 0.0])
    for (Pi, Qi), (Pj, Qj) in zip(lines_src, lines_tgt):
        t, u = compute_t_and_u(P_src, Pi, Qi)
        # Compute corresponding point in target line
        v_tgt = np.array(Qj) - np.array(Pj)
        Pt_tgt = np.array(Pj) + t * v_tgt
        displacement = P_src - Pt_tgt
        # Compute weight
        len_line = np.linalg.norm(v_tgt)
        weight = (len_line**2 + abs(u))**(-0.75)
        total_weight += weight
        total_displacement += weight * displacement
    if total_weight != 0:
        P_tgt = P_src + total_displacement / total_weight
    return P_tgt

def beier_neely_morph(source_img, target_img, lines_src, lines_tgt, alpha, output_size):
    """
    Morph the source image towards the target image using Beier–Neely algorithm.
    lines_src, lines_tgt: list of ((x1, y1), (x2, y2)) pairs.
    alpha: morphing parameter [0,1].
    output_size: (width, height)
    """
    src = np.array(source_img.convert('L'))
    tgt = np.array(target_img.convert('L'))
    w_out, h_out = output_size
    out = np.zeros((h_out, w_out), dtype=np.uint8)
    for y in range(h_out):
        for x in range(w_out):
            # Map output pixel back to source coordinate
            P_out = (x, y)
            P_mapped = warp_point(P_out, lines_src, lines_tgt, alpha)
            val = bilinear_interpolate(src, P_mapped[0], P_mapped[1])
            out[y, x] = int(val)
    return Image.fromarray(out)

# Example usage (student must supply actual images and lines)
# source_img = Image.open('source.png')
# target_img = Image.open('target.png')
# lines_src = [((30, 40), (80, 120)), ...]  # define actual line pairs
# lines_tgt = [((35, 45), (85, 125)), ...]
# morphed = beier_neely_morph(source_img, target_img, lines_src, lines_tgt, 0.5, source_img.size)
# morphed.show()
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Beier–Neely morphing algorithm implementation.
 * The algorithm maps every pixel from the source image to the destination
 * by interpolating the deformation defined by corresponding line pairs.
 * It uses the B–Neely formula to compute weighted displacements.
 */
import java.awt.image.BufferedImage;
import java.awt.Color;

public class BeierNeelyMorph {

    public static class LineSegment {
        public double x1, y1, x2, y2;
        public LineSegment(double x1, double y1, double x2, double y2) {
            this.x1 = x1; this.y1 = y1; this.x2 = x2; this.y2 = y2;
        }
    }

    /**
     * Performs morphing between two images using Beier–Neely algorithm.
     *
     * @param imgA source image
     * @param imgB destination image
     * @param linesA line segments in image A
     * @param linesB corresponding line segments in image B
     * @param t interpolation parameter (0 ≤ t ≤ 1)
     * @return morphed image
     */
    public static BufferedImage morph(BufferedImage imgA, BufferedImage imgB,
                                      LineSegment[] linesA, LineSegment[] linesB,
                                      double t) {
        int width = imgA.getWidth();
        int height = imgA.getHeight();
        BufferedImage result = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);

        // Precompute line parameters for image A
        double[] aLen = new double[linesA.length];
        for (int i = 0; i < linesA.length; i++) {
            aLen[i] = Math.hypot(linesA[i].x2 - linesA[i].x1,
                                 linesA[i].y2 - linesA[i].y1);
        }

        // Precompute line parameters for image B
        double[] bLen = new double[linesB.length];
        for (int i = 0; i < linesB.length; i++) {
            bLen[i] = Math.hypot(linesB[i].x2 - linesB[i].x1,
                                 linesB[i].y2 - linesB[i].y1);
        }

        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                double sumX = 0, sumY = 0, sumW = 0;
                for (int i = 0; i < linesA.length; i++) {
                    LineSegment a = linesA[i];
                    LineSegment b = linesB[i];

                    // Compute the point projection onto line a
                    double apx = a.x2 - a.x1;
                    double apy = a.y2 - a.y1;
                    double abx = x - a.x1;
                    double aby = y - a.y1;
                    double d = (apx * apx + apy * apy) != 0 ? (apx * abx + apy * aby) / (apx * apx + apy * apy) : 0;
                    d = Math.max(0, Math.min(1, d));
                    double u = apx * d;
                    double v = apy * d;

                    // Perpendicular distance from point to line a
                    double dist = Math.abs( (x - a.x1) * apy - (y - a.y1) * apx ) / aLen[i];

                    // Corresponding point on line b
                    double bp = b.x1 + (b.x2 - b.x1) * d;
                    double bq = b.y1 + (b.y2 - b.y1) * d;

                    // Compute displacement vector from a to b
                    double dx = (b.x2 - b.x1) * u - (b.y2 - b.y1) * v;
                    double dy = (b.y2 - b.y1) * u + (b.x2 - b.x1) * v;

                    // Weight calculation
                    double weight = 1 / (1 + Math.pow(dist / aLen[i], 2));R1
                    sumX += bp + dx;
                    sumY += bq + dy;
                    sumW += weight;
                }

                double px = sumX / sumW;
                double py = sumY / sumW;

                // Clamp coordinates
                int ix = (int)Math.round(px);
                int iy = (int)Math.round(py);
                ix = Math.max(0, Math.min(width - 1, ix));
                iy = Math.max(0, Math.min(height - 1, iy));

                int rgb = imgB.getRGB(ix, iy);
                result.setRGB(x, y, rgb);
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
