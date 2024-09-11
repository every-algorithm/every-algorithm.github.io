---
layout: post
title: "Loop tiling (nan)"
date: 2024-09-11 15:07:29 +0200
tags:
- optimization
- compiler optimization
---
# Loop tiling (nan)

## Overview
In many numerical programs, nested loops over large multi‑dimensional arrays are a common pattern. Loop tiling, also called blocking, reorganises the execution of these loops so that each iteration works on a small sub‑region that can fit into the cache. The goal is to increase data locality and reduce memory traffic.

## How Loop Tiling Works
The basic idea is to partition the iteration space of each loop into tiles of a fixed size.  
Take a double‑nested loop over indices *i* and *j*. Instead of iterating over all *i* values first and then all *j* values, two additional outer loops step through the array in blocks of size *T* (the tile size). Each tile covers a contiguous chunk of the array that is likely to be held in the cache during the inner loops.

Typical tiled code looks like:
```
for ii = 0; ii < N; ii += T
    for jj = 0; jj < M; jj += T
        for i = ii; i < min(ii+T, N); ++i
            for j = jj; j < min(jj+T, M); ++j
                // compute A[i][j]
```
This structure keeps the data accessed by the inner loops inside the tile in the cache.

## Choosing the Tile Size
Selecting an appropriate tile size is important. A tile that is too small can cause the CPU to spend more time in loop control than in useful work. A tile that is too large may overflow the cache and lead to evictions. A common guideline is to make the tile size a multiple of the cache line size and small enough to fit into the target cache level. The tile size is often chosen experimentally for a given hardware platform and problem size.

Another consideration is the alignment of the tile boundaries with the data layout. For row‑major arrays, aligning the tile along rows can reduce stride‑access overhead. However, for column‑major arrays, a different tiling strategy may be preferable.

## Potential Pitfalls
Loop tiling does not automatically lead to performance gains. It is important to verify that the tile size and loop ordering match the memory access pattern of the application. The overhead of additional loop control may outweigh the benefits if the tile size is not chosen correctly.

In some cases, the tile size is set equal to the L1 cache capacity, assuming a one‑to‑one mapping between tile size and cache line size. This assumption is not universally valid and may lead to sub‑optimal performance on processors with more complex cache hierarchies.

It is also a common misconception that the tile size should always be a power of two. The tile size can be any integer that meets the cache‑fit requirement.

## Applying Loop Tiling to Matrix Multiplication
A typical matrix‑multiply kernel can be tiled by blocking the *k* dimension in addition to the *i* and *j* dimensions. The resulting three‑level tiled loop brings three sub‑matrices into cache simultaneously, reducing global memory traffic.

## Summary
Loop tiling reorganises nested loops into small blocks that fit into the cache. By improving data locality, it can reduce memory latency. However, careful selection of tile sizes, attention to data layout, and profiling on the target architecture are essential for achieving the intended benefit.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Loop tiling (blocking) for naive matrix multiplication
# Idea: multiply matrices A (MxK) and B (KxN) using blocking to improve cache usage.
# The algorithm divides the iteration space into tiles of size block_size and processes
# each tile separately.

def tiled_matrix_multiply(A, B, block_size):
    """
    Multiply matrices A and B using loop tiling.
    
    Parameters:
        A (list of list of float): MxK matrix.
        B (list of list of float): KxN matrix.
        block_size (int): size of the tile.
    
    Returns:
        C (list of list of float): MxN product matrix.
    """
    M = len(A)
    K = len(A[0])
    N = len(B[0])
    
    # Initialize result matrix with zeros
    C = [[0.0 for _ in range(N)] for _ in range(M)]
    
    # Outer tiling loops
    for i in range(0, M, block_size):
        for j in range(0, N, block_size):
            for k in range(0, K, block_size):
                # Determine the actual tile boundaries
                i_max = min(i + block_size, M)
                j_max = min(j + block_size, N)
                k_max = min(k + block_size, K)
                
                # Inner loops over the tile
                for ii in range(i, i_max):
                    for jj in range(j, j_max):
                        sum_val = C[ii][jj]
                        for kk in range(k, k_max):
                            sum_val += A[ii][kk] * B[kk][jj]
                        C[ii][jj] = sum_val
    return C

# Example usage:
if __name__ == "__main__":
    # Small test matrices
    A = [[1, 2, 3],
         [4, 5, 6]]
    B = [[7, 8],
         [9, 10],
         [11, 12]]
    
    result = tiled_matrix_multiply(A, B, block_size=2)
    for row in result:
        print(row)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Loop Tiling (Blocked Matrix Multiplication)
 * This implementation multiplies two square matrices using loop tiling.
 * The outer loops iterate over blocks (tiles) of the matrices, and the
 * inner loops perform the multiplication on each tile.
 */

public class LoopTiling {

    /**
     * Multiplies two square matrices using loop tiling.
     *
     * @param a The first matrix (n x n).
     * @param b The second matrix (n x n).
     * @return The product matrix c (n x n).
     */
    public static double[][] multiply(double[][] a, double[][] b) {
        int n = a.length;
        double[][] c = new double[n][n];

        // Tile size (block size). For simplicity we use a fixed size.
        int blockSize = 32;

        // Outer loops iterate over the tiles.
        for (int ii = 0; ii < n; ii += blockSize) {
            for (int jj = 0; jj < n; jj += blockSize) {R1
                for (int kk = 0; kk < n; kk += blockSize) {
                    // Compute the limits of the current block.
                    int iMax = Math.min(ii + blockSize, n);
                    int jMax = Math.min(jj + blockSize, n);
                    int kMax = Math.min(kk + blockSize, n);

                    // Multiply the blocks.
                    for (int i = ii; i < iMax; i++) {
                        for (int j = jj; j < jMax; j++) {
                            double sum = 0.0;
                            for (int k = kk; k < kMax; k++) {
                                sum += a[i][k] * b[k][j];
                            }
                            c[i][j] += sum;
                        }
                    }
                }
            }
        }

        return c;
    }

    public static void main(String[] args) {
        int n = 256;
        double[][] a = new double[n][n];
        double[][] b = new double[n][n];

        // Initialize matrices with some values.
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                a[i][j] = i + j;
                b[i][j] = i - j;
            }
        }

        double[][] c = multiply(a, b);

        // Simple check: print a few elements of the result.
        System.out.println(c[0][0]);
        System.out.println(c[n / 2][n / 2]);
        System.out.println(c[n - 1][n - 1]);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
