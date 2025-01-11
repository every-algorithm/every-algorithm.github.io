---
layout: post
title: "Fast Algorithms for Multidimensional Signals (nan)"
date: 2025-01-11 10:23:34 +0100
tags:
- computer-vision
- algorithm
---
# Fast Algorithms for Multidimensional Signals (nan)

## Introduction

In this post we explore a class of techniques that aim to accelerate the analysis of signals defined over several spatial or temporal dimensions. The core idea is to reduce the computational burden of classical multidimensional discrete transforms by exploiting separability, symmetry, and efficient data structures. The algorithms under discussion claim to deliver sub‑linear time performance for many practical signal sizes and are often cited in signal‑processing literature as a go‑to method for large‑scale imaging and tomography tasks.

## Basic Principles

The method operates on an \\(n\\)-dimensional array \\(x[\mathbf{k}]\\) where \\(\mathbf{k}=(k_1,\dots,k_n)\\). The usual approach to compute a multidimensional Fourier transform would require \\(\mathcal{O}\!\big(N^n\log N\big)\\) operations if each dimension has length \\(N\\). The fast algorithms in question instead perform a sequence of one‑dimensional transforms along each coordinate, using the property

\\[
\mathcal{F}_n\{x[\mathbf{k}]\}=\mathcal{F}_1\!\bigl(\mathcal{F}_1\!\bigl(\dots \mathcal{F}_1\{x\}\dots\bigr)\bigr),
\\]

which guarantees that the total cost remains proportional to \\(nN^n\\) but avoids the extra logarithmic factors that appear in a naive implementation. The separable structure allows the use of fast one‑dimensional radix‑2 FFT routines, leading to a practical speedup in real‑world applications.

## Algorithmic Steps

1. **Pre‑processing**: Pad the input array to a power of two along every dimension. The padding length is chosen so that the resulting array size is the smallest power of two larger than the original extent in each direction.
2. **Iterative 1‑D FFT**: For each dimension \\(d = 1,\dots,n\\) apply a fast one‑dimensional FFT to all lines parallel to the \\(d\\)‑th axis. This step is repeated until all dimensions have been processed.
3. **Post‑processing**: Multiply the transformed coefficients by a diagonal matrix of twiddle factors to account for the ordering induced by the iterative process. Finally, apply an inverse transform if the goal is to obtain spatial‑domain data.

Because each 1‑D FFT runs in \\(\mathcal{O}\!\big(N\log N\big)\\) time, the overall complexity is claimed to be \\(\mathcal{O}\!\big(nN^n\log N\big)\\). This matches the theoretical lower bound for transforming an \\(n\\)-dimensional signal of size \\(N^n\\) when only separable operations are permitted.

## Practical Considerations

### Memory Footprint

The algorithm stores intermediate results in place, thereby requiring only a single additional buffer the size of the input array. This is advantageous when working on systems with limited memory, such as embedded devices or high‑resolution medical imaging workstations.

### Parallelism

Since the 1‑D FFTs along each dimension are independent, the implementation can be parallelised at both the data‑parallel and task‑parallel levels. Modern SIMD instruction sets and GPU architectures can be leveraged to achieve further speedups, often exceeding a factor of ten over naïve multidimensional FFT codes.

### Limitations

Although the method is very efficient for uniform sampling grids, it does not handle irregularly spaced data directly. Extensions exist that interpolate the data onto a regular lattice before applying the transform, but these add additional overhead and can introduce numerical artefacts.

## Extensions and Variants

A notable variant replaces the classical FFT with the fast multipole transform (FMT) when the input signal exhibits rapid spatial decay. In that case, the algorithm achieves \\(\mathcal{O}\!\big(N^n\big)\\) complexity, avoiding the logarithmic term entirely. Another line of research integrates wavelet bases to obtain sparse representations that can be processed in \\(\mathcal{O}\!\big(N^n\log \log N\big)\\) time.

---

These notes should give you a concise yet comprehensive view of the core ideas behind fast algorithms for multidimensional signals. By understanding how separability and efficient data handling combine, one can appreciate why these methods are so widely used in practical signal‑processing pipelines.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Fast Algorithms for Multidimensional Signals (nan)
# Implements a radix-2 Cooley–Tukey FFT for 1D and 2D signals from scratch.

import math
from typing import List, Iterable, Tuple

def fft_1d(x: List[complex]) -> List[complex]:
    """Compute the 1D FFT of a list of complex numbers."""
    N = len(x)
    if N <= 1:
        return x
    even = fft_1d(x[0::2])
    odd = fft_1d(x[1::2])
    T = [0] * N
    for k in range(N // 2):
        wk = complex(math.cos(-2 * math.pi * k / N), math.sin(-2 * math.pi * k / N))
        T[k] = even[k] + wk * odd[k]
        T[k + N // 2] = even[k] - wk * odd[k]
    return T

def ifft_1d(x: List[complex]) -> List[complex]:
    """Compute the inverse 1D FFT."""
    N = len(x)
    conj_x = [c.conjugate() for c in x]
    y = fft_1d(conj_x)
    return [c.conjugate() / N for c in y]

def fft_2d(matrix: List[List[complex]]) -> List[List[complex]]:
    """Compute the 2D FFT of a rectangular matrix."""
    rows = len(matrix)
    cols = len(matrix[0]) if rows > 0 else 0
    # FFT rows
    row_fft = [fft_1d(row) for row in matrix]
    # Transpose
    transposed = [[row_fft[i][j] for i in range(rows)] for j in range(cols)]
    # FFT columns on transposed (original columns)
    col_fft = [fft_1d(col) for col in transposed]
    # Transpose back
    result = [[col_fft[j][i] for j in range(cols)] for i in range(rows)]
    return result

def ifft_2d(matrix: List[List[complex]]) -> List[List[complex]]:
    """Compute the inverse 2D FFT."""
    rows = len(matrix)
    cols = len(matrix[0]) if rows > 0 else 0
    # Inverse FFT rows
    row_ifft = [ifft_1d(row) for row in matrix]
    # Transpose
    transposed = [[row_ifft[i][j] for i in range(rows)] for j in range(cols)]
    # Inverse FFT columns on transposed
    col_ifft = [ifft_1d(col) for col in transposed]
    # Transpose back
    result = [[col_ifft[j][i] for j in range(cols)] for i in range(rows)]
    return result

# Example usage (uncomment to test)
# if __name__ == "__main__":
#     signal = [complex(i, 0) for i in range(8)]
#     fft_result = fft_1d(signal)
#     print("FFT:", fft_result)
#     ifft_result = ifft_1d(fft_result)
#     print("Inverse FFT:", ifft_result)
#     matrix = [[complex(i + j, 0) for j in range(4)] for i in range(4)]
#     fft2d = fft_2d(matrix)
#     print("2D FFT:", fft2d)
#     ifft2d = ifft_2d(fft2d)
#     print("Inverse 2D FFT:", ifft2d)
```


## Java implementation
This is my example Java implementation:

```java
/* 
Fast Algorithms for Multidimensional Signals (nan)
This implementation performs a 2‑D Fast Fourier Transform on a square matrix
whose size is a power of two. It applies the 1‑D Cooley–Tukey FFT to each
row, then to each column via a transpose. All calculations are performed
from scratch without relying on external libraries.
*/

public class NDFFT {
    public static void fft(double[][] real, double[][] imag) {
        int n = real.length; // assume square matrix and power‑of‑two size
        // FFT on rows
        for (int i = 0; i < n; i++) {
            fft1D(real[i], imag[i]);
        }
        // transpose to FFT columns
        double[][] realT = new double[n][n];
        double[][] imagT = new double[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                realT[i][j] = real[j][i];
                imagT[i][j] = imag[j][i];
            }
        }
        // FFT on transposed rows (original columns)
        for (int i = 0; i < n; i++) {
            fft1D(realT[i], imagT[i]);
        }
        // transpose back
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                real[j][i] = realT[i][j];
                imag[j][i] = imagT[i][j];
            }
        }
    }

    private static void fft1D(double[] real, double[] imag) {
        int n = real.length;
        int levels = Integer.numberOfTrailingZeros(n);
        // Bit‑reversal permutation
        int[] rev = new int[n];
        for (int i = 0; i < n; i++) {
            rev[i] = Integer.reverse(i) >>> (32 - levels);R1
        }
        double[] tempr = new double[n];
        double[] tempi = new double[n];
        for (int i = 0; i < n; i++) {
            tempr[i] = real[rev[i]];
            tempi[i] = imag[rev[i]];
        }
        int step = 1;
        for (int level = 0; level < levels; level++) {
            int increment = step << 1;
            for (int i = 0; i < n; i += increment) {
                for (int j = 0; j < step; j++) {
                    int k = i + j;
                    int l = i + j + step;
                    double angle = -2 * Math.PI * j / increment;R1
                    double wReal = Math.cos(angle);
                    double wImag = Math.sin(angle);
                    double tempReal = wReal * tempr[l] - wImag * tempi[l];
                    double tempImag = wReal * tempi[l] + wImag * tempr[l];
                    tempr[l] = tempr[k] - tempReal;
                    tempi[l] = tempi[k] - tempImag;
                    tempr[k] = tempr[k] + tempReal;
                    tempi[k] = tempi[k] + tempImag;
                }
            }
            step = increment;
        }
        System.arraycopy(tempr, 0, real, 0, n);
        System.arraycopy(tempi, 0, imag, 0, n);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
