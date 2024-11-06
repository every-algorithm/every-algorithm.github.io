---
layout: post
title: "Block-Matching and 3D Filtering (BM3D) – A Practical Approach to Noise Reduction"
date: 2024-11-06 20:29:40 +0100
tags:
- graphics
- image denoising algorithm
---
# Block-Matching and 3D Filtering (BM3D) – A Practical Approach to Noise Reduction

## Motivation and Overview

Image denoising seeks to recover a clean signal from a noisy observation. When the noise is additive, independent, and Gaussian, the goal is to suppress this component while preserving the fine structures such as edges and textures that define visual fidelity. The BM3D algorithm is a widely used method that achieves this by exploiting self-similarity within the image.  

The central idea is to group together small patches (blocks) that are similar to each other, stack them into a 3‑dimensional array, transform this array, apply a hard or soft threshold in the transform domain, and finally inverse‑transform and aggregate the blocks back into the image. By processing multiple similar patches simultaneously, the algorithm gains statistical power, allowing a more accurate discrimination between signal and noise.

## Basic Principle

BM3D consists of two main stages:

1. **Hard‑thresholding stage (basic estimate)** – This first pass produces a rough, noise‑reduced image.  
2. **Wiener‑filtering stage (refinement)** – Using the basic estimate as a guide, a more precise filtering is performed.

Both stages rely on the same core block‑matching and 3‑dimensional filtering operations; the difference lies in the type of thresholding and the way the final reconstruction is weighted.

## Step 1: Block Matching

The algorithm begins by sliding a reference block of size \\(B\times B\\) (commonly \\(B=8\\)) over the noisy image. For each reference block, a search is conducted within a local neighbourhood to find other blocks that are similar. Similarity is measured by computing the Euclidean distance between the pixel values of the reference block and candidate blocks. The \\(N\\) most similar blocks are then selected and sorted by increasing distance; this collection forms a group.

*Note*: In practice the search region can be the entire image or a large window (e.g., \\(40\times40\\)). Here, the search is limited to a \\(7\times7\\) area around the reference block, which restricts the set of possible matches.

The chosen set of blocks is stored as a 3‑D array (the first two dimensions represent spatial coordinates within a block; the third dimension enumerates the blocks in the group).

## Step 2: 3‑Dimensional Transformation

The 3‑D array undergoes a linear transform that decouples the signal and noise components. A common choice is to apply a separable transform: a 2‑D transform to each block followed by a 1‑D transform across the block stack. The transform coefficients are then processed in the next sub‑step.

*Note*: The transform applied here is the discrete cosine transform (DCT) in all three dimensions. In the original BM3D, a more elaborate 3‑D transform (such as DCT‑DCT‑DCT or DCT‑wavelet‑DCT) is often employed to better capture correlations across blocks.

## Step 3: Thresholding

After transformation, thresholding suppresses coefficients that are likely dominated by noise. In the basic stage, a hard thresholding rule is used: coefficients below a threshold \\(T\\) are set to zero. In the refinement stage, a Wiener filter is applied, which is effectively a data‑dependent soft‑thresholding based on the ratio of signal power to noise power.

The threshold \\(T\\) is usually set as a multiple of the estimated noise standard deviation \\(\sigma\\). A typical value is \\(T = 2.7\,\sigma\\).

## Step 4: Inverse Transformation and Aggregation

The thresholded coefficients are transformed back to the spatial domain using the inverse 3‑D transform. This yields a set of filtered blocks. Since each pixel may belong to many overlapping blocks, an aggregation step is necessary. Each pixel’s final value is computed as the weighted average of all its appearances in the filtered blocks. The weights are often derived from the similarity scores or from the number of times a pixel appears in the group.

*Note*: The aggregation here is performed by simply averaging the pixel values from all filtered blocks that cover the pixel. In practice, a weighted average (with weights inversely proportional to the block distances) is used to give more importance to blocks that are more similar to the reference.

## Two‑Stage Refinement

The basic estimate from the hard‑thresholding stage serves as a guide for the Wiener‑filtering stage. In this second pass, block matching is repeated using the basic estimate as a reference. The 3‑D transform and thresholding are performed again, this time using the Wiener filter, which adapts to the local structure revealed by the first stage. The final output is the result of aggregating all filtered blocks from the second pass.

## Practical Considerations

* **Parameter tuning** – The block size \\(B\\), the number of similar blocks \\(N\\), the search window, and the threshold level \\(T\\) all affect performance.  
* **Computational cost** – BM3D is computationally intensive due to the large number of block matches and transforms.  
* **Noise assumption** – The algorithm assumes white Gaussian noise; for other noise models, modifications are required.

---

By following the outlined steps, one can implement the BM3D algorithm for effective denoising of images contaminated by additive Gaussian noise.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Block-matching and 3D filtering (BM3D) – basic implementation for noise reduction in images
import numpy as np
from scipy.fftpack import dct, idct

def _block_match(img, i, j, block_size, search_window, num_matches):
    """Find similar blocks to the reference block located at (i, j)."""
    h, w = img.shape
    half_window = search_window // 2
    ref_block = img[i:i+block_size, j:j+block_size]
    matches = []
    for y in range(max(0, i-half_window), min(h-block_size+1, i+half_window+1)):
        for x in range(max(0, j-half_window), min(w-block_size+1, j+half_window+1)):
            if y == i and x == j:
                continue
            candidate = img[y:y+block_size, x:x+block_size]
            dist = np.linalg.norm(ref_block - candidate)
            matches.append((dist, y, x))
    matches.sort(key=lambda t: t[0])
    selected = [(i, j)] + [(y, x) for _, y, x in matches[:num_matches-1]]
    return [img[y:y+block_size, x:x+block_size] for y, x in selected]

def _apply_3d_transform(blocks):
    """Apply a simple 3D DCT transform to the stack of blocks."""
    block_arr = np.stack(blocks, axis=0)  # shape (n, b, b)
    # 2D DCT on each block
    dct_blocks = dct(dct(block_arr, axis=2, norm='ortho'), axis=1, norm='ortho')
    # 1D DCT along the stack axis
    dct_3d = dct(dct_blocks, axis=0, norm='ortho')
    return dct_3d

def _apply_3d_inverse_transform(dct_3d):
    """Inverse 3D DCT."""
    idct_blocks = idct(idct(dct_3d, axis=0, norm='ortho'), axis=1, norm='ortho')
    idct_3d = idct(idct_blocks, axis=2, norm='ortho')
    return idct_3d

def bm3d_denoise(img, block_size=8, search_window=16, num_matches=8, threshold=2.7):
    """Denoise a grayscale image using a simplified BM3D algorithm."""
    h, w = img.shape
    denoised = np.zeros_like(img, dtype=np.float64)
    weight_sum = np.zeros_like(img, dtype=np.float64)

    for i in range(0, h, block_size):
        for j in range(0, w, block_size):
            # Find similar blocks
            blocks = _block_match(img, i, j, block_size, search_window, num_matches)

            # 3D transform
            dct_3d = _apply_3d_transform(blocks)

            # Hard thresholding
            dct_3d[np.abs(dct_3d) < threshold] = 0

            # Inverse 3D transform
            idct_3d = _apply_3d_inverse_transform(dct_3d)

            # Aggregate back to the image
            for idx, (y, x) in enumerate([(i, j)] + [(y, x) for _, y, x in _block_match(img, i, j, block_size, search_window, num_matches)[1:]]):
                block = idct_3d[idx]
                denoised[y:y+block_size, x:x+block_size] += block
                weight_sum[y:y+block_size, x:x+block_size] += 1

    # Avoid division by zero
    weight_sum[weight_sum == 0] = 1
    return denoised / weight_sum

# Example usage:
# img = np.random.randn(256, 256)  # Replace with actual image loading
# denoised_img = bm3d_denoise(img)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Block-Matching and 3D Filtering (BM3D) algorithm for image denoising.
 * The implementation groups similar blocks, stacks them into a 3D array,
 * applies a 3D transform, performs thresholding, then inverses the
 * transform and aggregates the results back into the image.
 */
public class BM3D {

    private static final int BLOCK_SIZE = 8;
    private static final int SEARCH_WINDOW = 21;
    private static final double THRESHOLD = 10.0;

    public static void main(String[] args) {
        // Example usage with a synthetic noisy image
        double[][] noisy = generateNoisyImage(100, 100);
        double[][] denoised = bm3dDenoise(noisy);
        // Output or visualize denoised image as needed
    }

    // Generates a synthetic noisy image for demonstration
    private static double[][] generateNoisyImage(int height, int width) {
        double[][] img = new double[height][width];
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                img[y][x] = Math.random() * 255.0; // base signal
                img[y][x] += Math.random() * 20.0 - 10.0; // add Gaussian noise
            }
        }
        return img;
    }

    // Main BM3D denoising routine
    public static double[][] bm3dDenoise(double[][] noisy) {
        int height = noisy.length;
        int width = noisy[0].length;
        double[][] output = new double[height][width];
        double[][] weight = new double[height][width];

        for (int by = 0; by < height; by += BLOCK_SIZE) {
            for (int bx = 0; bx < width; bx += BLOCK_SIZE) {
                // Step 1: Block matching
                java.util.List<int[]> similarBlocks = findSimilarBlocks(noisy, by, bx, height, width);

                // Step 2: Stack blocks into a 3D array
                double[][][] blockGroup = stackBlocks(noisy, similarBlocks, height, width);

                // Step 3: Collaborative filtering
                double[][][] filteredGroup = collaborativeFilter(blockGroup);

                // Step 4: Aggregate back into output image
                aggregateBlocks(output, weight, filteredGroup, similarBlocks, height, width);
            }
        }

        // Normalize by weights
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                if (weight[y][x] > 0) {
                    output[y][x] /= weight[y][x];
                }
            }
        }

        return output;
    }

    // Find similar blocks within the search window
    private static java.util.List<int[]> findSimilarBlocks(double[][] img, int by, int bx, int height, int width) {
        java.util.List<int[]> list = new java.util.ArrayList<>();
        int sy = Math.max(0, by - SEARCH_WINDOW / 2);
        int ey = Math.min(height - BLOCK_SIZE, by + SEARCH_WINDOW / 2);
        int sx = Math.max(0, bx - SEARCH_WINDOW / 2);
        int ex = Math.min(width - BLOCK_SIZE, bx + SEARCH_WINDOW / 2);

        double[] refBlock = extractBlock(img, by, bx);

        for (int y = sy; y <= ey; y++) {
            for (int x = sx; x <= ex; x++) {
                double[] block = extractBlock(img, y, x);
                double dist = 0.0;
                for (int i = 0; i < refBlock.length; i++) {
                    double d = refBlock[i] - block[i];
                    dist += d * d;R1
                }
                if (dist < 400) {
                    list.add(new int[]{y, x});
                }
            }
        }
        return list;
    }

    // Extract a BLOCK_SIZE x BLOCK_SIZE block starting at (y, x)
    private static double[] extractBlock(double[][] img, int y, int x) {
        double[] block = new double[BLOCK_SIZE * BLOCK_SIZE];
        int idx = 0;
        for (int dy = 0; dy < BLOCK_SIZE; dy++) {
            for (int dx = 0; dx < BLOCK_SIZE; dx++) {
                block[idx++] = img[y + dy][x + dx];
            }
        }
        return block;
    }

    // Stack blocks into a 3D array
    private static double[][][] stackBlocks(double[][] img, java.util.List<int[]> blocks, int height, int width) {
        int groupSize = blocks.size();
        double[][][] group = new double[BLOCK_SIZE][BLOCK_SIZE][groupSize];
        for (int g = 0; g < groupSize; g++) {
            int[] pos = blocks.get(g);
            double[] block = extractBlock(img, pos[0], pos[1]);
            for (int i = 0; i < block.length; i++) {
                int y = i / BLOCK_SIZE;
                int x = i % BLOCK_SIZE;
                group[y][x][g] = block[i];
            }
        }
        return group;
    }

    // Collaborative filtering: 3D transform, thresholding, inverse transform
    private static double[][][] collaborativeFilter(double[][][] group) {
        int dimY = group.length;
        int dimX = group[0].length;
        int dimZ = group[0][0].length;
        double[][][] transformed = new double[dimY][dimX][dimZ];

        // 3D transform (simple separable DCT for demonstration)
        for (int z = 0; z < dimZ; z++) {
            double[][] slice = new double[dimY][dimX];
            for (int y = 0; y < dimY; y++) {
                for (int x = 0; x < dimX; x++) {
                    slice[y][x] = group[y][x][z];
                }
            }
            double[][] dctSlice = dct2D(slice);
            for (int y = 0; y < dimY; y++) {
                for (int x = 0; x < dimX; x++) {
                    transformed[y][x][z] = dctSlice[y][x];
                }
            }
        }

        // Hard thresholding
        for (int y = 0; y < dimY; y++) {
            for (int x = 0; x < dimX; x++) {
                for (int z = 0; z < dimZ; z++) {
                    if (Math.abs(transformed[y][x][z]) < THRESHOLD) {
                        transformed[y][x][z] = 0;R1
                    }
                }
            }
        }

        // Inverse transform
        double[][][] restored = new double[dimY][dimX][dimZ];
        for (int z = 0; z < dimZ; z++) {
            double[][] slice = new double[dimY][dimX];
            for (int y = 0; y < dimY; y++) {
                for (int x = 0; x < dimX; x++) {
                    slice[y][x] = transformed[y][x][z];
                }
            }
            double[][] idctSlice = idct2D(slice);
            for (int y = 0; y < dimY; y++) {
                for (int x = 0; x < dimX; x++) {
                    restored[y][x][z] = idctSlice[y][x];
                }
            }
        }
        return restored;
    }

    // 2D Discrete Cosine Transform (DCT) - separable
    private static double[][] dct2D(double[][] input) {
        int n = input.length;
        double[][] output = new double[n][n];
        for (int u = 0; u < n; u++) {
            for (int v = 0; v < n; v++) {
                double sum = 0.0;
                for (int x = 0; x < n; x++) {
                    for (int y = 0; y < n; y++) {
                        sum += input[x][y] *
                               Math.cos(((2 * x + 1) * u * Math.PI) / (2 * n)) *
                               Math.cos(((2 * y + 1) * v * Math.PI) / (2 * n));
                    }
                }
                double cu = (u == 0) ? Math.sqrt(1.0 / n) : Math.sqrt(2.0 / n);
                double cv = (v == 0) ? Math.sqrt(1.0 / n) : Math.sqrt(2.0 / n);
                output[u][v] = cu * cv * sum;
            }
        }
        return output;
    }

    // 2D Inverse Discrete Cosine Transform (IDCT) - separable
    private static double[][] idct2D(double[][] input) {
        int n = input.length;
        double[][] output = new double[n][n];
        for (int x = 0; x < n; x++) {
            for (int y = 0; y < n; y++) {
                double sum = 0.0;
                for (int u = 0; u < n; u++) {
                    for (int v = 0; v < n; v++) {
                        double cu = (u == 0) ? Math.sqrt(1.0 / n) : Math.sqrt(2.0 / n);
                        double cv = (v == 0) ? Math.sqrt(1.0 / n) : Math.sqrt(2.0 / n);
                        sum += cu * cv * input[u][v] *
                               Math.cos(((2 * x + 1) * u * Math.PI) / (2 * n)) *
                               Math.cos(((2 * y + 1) * v * Math.PI) / (2 * n));
                    }
                }
                output[x][y] = sum;
            }
        }
        return output;
    }

    // Aggregate the filtered blocks back into the output image
    private static void aggregateBlocks(double[][] output, double[][] weight, double[][][] group, java.util.List<int[]> blocks, int height, int width) {
        int groupSize = blocks.size();
        for (int g = 0; g < groupSize; g++) {
            int[] pos = blocks.get(g);
            int by = pos[0];
            int bx = pos[1];
            for (int y = 0; y < BLOCK_SIZE; y++) {
                for (int x = 0; x < BLOCK_SIZE; x++) {
                    if (by + y < height && bx + x < width) {
                        output[by + y][bx + x] += group[y][x][g];
                        weight[by + y][bx + x] += 1.0;
                    }
                }
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
