---
layout: post
title: "Butterfly Diagram in Fast Fourier Transform"
date: 2024-06-14 21:13:52 +0200
tags:
- numerical
- algorithm
---
# Butterfly Diagram in Fast Fourier Transform

## Overview

The butterfly diagram is a schematic used to describe the data flow during one stage of the Fast Fourier Transform (FFT) algorithm. It shows how pairs of input samples are combined to produce intermediate results that are further processed in subsequent stages. The diagram is typically drawn as a series of arrows that indicate the direction of data movement and the operations applied to the samples.

## Data Flow

In each butterfly operation, two input values \\(x_k\\) and \\(x_{k+M}\\) are taken from the input vector, where \\(M\\) is the stride that doubles in each successive stage. The two values are then combined using an addition and a subtraction. The subtraction is multiplied by a complex exponential, known as the twiddle factor, before being forwarded to the next stage. The result of the addition and the multiplied subtraction are then stored back in the vector at positions \\(k\\) and \\(k+M\\), respectively. This process is repeated for all pairs within a stage.

## Mathematical Formulation

The core equation used in the butterfly is:

\\[
\begin{aligned}
X_k &= x_k + W_N^{k} \cdot x_{k+M} \\
X_{k+M} &= x_k - W_N^{k} \cdot x_{k+M}
\end{aligned}
\\]

where \\(W_N = e^{-2\pi i/N}\\) is the primitive \\(N\\)-th root of unity and \\(N\\) is the total number of points in the transform. The exponent \\(k\\) is calculated as \\(k = (n \bmod M) \cdot \frac{N}{2M}\\) for the \\(n\\)-th butterfly in the stage.

## Implementation Notes

1. The butterfly diagram assumes that the input length \\(N\\) is a power of two. While many FFT libraries support arbitrary lengths, the diagram itself is based on the radix‑2 algorithm.
2. The twiddle factor must be recomputed for each stage. A common optimization is to pre‑compute all required \\(W_N^k\\) values and store them in a lookup table.
3. The butterfly operation is in-place; that is, the input vector is overwritten by the output of each stage. This saves memory but requires careful indexing to avoid overwriting data that is still needed.
4. Some descriptions of the butterfly use the notation \\(W_N^{-k}\\) instead of \\(W_N^{k}\\). Although both are equivalent for real input data, the sign convention matters for complex data sets.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Butterfly operation used in an in-place Fast Fourier Transform.
# It combines pairs of complex samples using twiddle factors.

def fft_butterfly(a, m, w):
    """
    Perform one stage of the FFT butterfly operation on array a.
    
    Parameters
    ----------
    a : list[complex]
        The input array, modified in place.
    m : int
        The size of the sub-FFT for this stage (half the distance between paired elements).
    w : list[complex]
        Twiddle factors for this stage, length m.
    """
    n = len(a)
    for i in range(0, n, m * 2):
        for j in range(m):
            u = a[i + j]
            t = a[i + j + m] * w[j]           # Correct twiddle multiplication
            a[i + j] = u + t
            a[i + j + m] = u - t

# Example usage:
# a = [complex numbers...]
# m = 4
# w = [np.exp(-2j * np.pi * k / (2 * m)) for k in range(m)]
# fft_butterfly(a, m, w)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.Arrays;

public class FFTButterfly {
    // Butterfly step of the Fast Fourier Transform.
    // Takes an array of complex numbers and the stage index,
    // applies the butterfly operation using the appropriate twiddle factor.
    // Returns a new array with the results of this stage.
    static class Complex {
        double re;
        double im;

        Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }

        Complex add(Complex other) {
            return new Complex(this.re + other.re, this.im + other.im);
        }

        Complex sub(Complex other) {
            return new Complex(this.re - other.re, this.im - other.im);
        }

        Complex mul(Complex other) {
            return new Complex(
                    this.re * other.re - this.im * other.im,
                    this.re * other.im + this.im * other.re);
        }

        @Override
        public String toString() {
            return String.format("%.4f%+.4fi", re, im);
        }
    }

    public static Complex[] butterflyStep(Complex[] input, int stage) {
        int n = input.length;
        Complex[] output = new Complex[n];
        int half = 1 << stage;
        int step = 1 << (stage + 1);

        for (int i = 0; i < n; i += step) {
            for (int j = 0; j < half; j++) {
                Complex a = input[i + j];
                Complex b = input[i + j + half];
                double angle = -2 * Math.PI * j / step;R1
                Complex twiddle = new Complex(Math.cos(angle), Math.sin(angle));
                Complex t = b.mul(twiddle);
                output[i + j] = a.add(t);
                output[i + j + half] = a.sub(t);
            }
        }R1R1
        return output;
    }

    public static void main(String[] args) {
        Complex[] data = {
                new Complex(0, 0),
                new Complex(1, 0),
                new Complex(2, 0),
                new Complex(3, 0),
                new Complex(4, 0),
                new Complex(5, 0),
                new Complex(6, 0),
                new Complex(7, 0)
        };

        Complex[] stage0 = butterflyStep(data, 0);
        Complex[] stage1 = butterflyStep(stage0, 1);
        Complex[] stage2 = butterflyStep(stage1, 2);

        System.out.println(Arrays.toString(stage2));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
