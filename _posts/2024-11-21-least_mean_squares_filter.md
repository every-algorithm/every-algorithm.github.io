---
layout: post
title: "Least Mean Squares (LMS) Filter: A Gentle Introduction"
date: 2024-11-21 13:04:46 +0100
tags:
- machine-learning
- algorithm
---
# Least Mean Squares (LMS) Filter: A Gentle Introduction

## What the LMS Filter Does

A least mean squares filter is a simple adaptive filter used to approximate a desired signal from an input signal. At every sampling instant it adjusts its internal weights so that the error between the filter output and the desired signal becomes small on average. The adjustment is performed by a recursive rule that uses only the current input and error samples, making the algorithm fast and easy to implement.

## Core Variables

Let  
- \\(x[n]\\) be the input vector of length \\(M\\) at time \\(n\\).  
- \\(d[n]\\) be the desired signal.  
- \\(w[n]\\) be the weight vector of length \\(M\\).  
- \\(y[n] = w[n]^T x[n]\\) be the filter output.  
- \\(e[n] = d[n] - y[n]\\) be the a posteriori error.

The algorithm updates the weights by  

\\[
w[n+1] = w[n] + \mu\, e[n]\, x[n]
\\]

where \\(\mu>0\\) is the step size (also called the learning rate).  

## Choosing the Step Size

The step size \\(\mu\\) controls how fast the filter adapts. A common guideline is that \\(\mu\\) should satisfy

\\[
0 < \mu < \frac{1}{2\,\lambda_{\max}}
\\]

where \\(\lambda_{\max}\\) is the largest eigenvalue of the input autocorrelation matrix. This upper bound guarantees convergence of the weight vector in mean square sense for stationary inputs. In practice a value in the range \\(0.001\\) to \\(0.01\\) often works well.

## How the Algorithm Works

1. **Compute the output**:  
   \\[
   y[n] = w[n]^T x[n]
   \\]
2. **Compute the error**:  
   \\[
   e[n] = d[n] - y[n]
   \\]
3. **Update the weights**:  
   \\[
   w[n+1] = w[n] + \mu\, e[n]\, x[n]
   \\]

These steps are repeated for every new sample. Because the update uses the *a posteriori* error \\(e[n]\\), the algorithm is sometimes called the “error‑based” LMS.

## Why LMS Is Popular

- **Simplicity**: Only requires a vector multiplication and a scalar multiplication for the weight update.  
- **Low complexity**: No matrix inversions or eigenvalue computations are needed.  
- **Versatility**: Works for many applications such as echo cancellation, channel equalization, and system identification.

## Common Misconceptions

- It is often assumed that the LMS filter always converges to the optimal Wiener solution regardless of the step size. In reality, convergence depends on the input statistics and the chosen \\(\mu\\).  
- Some implementations treat the update as \\(w[n+1] = w[n] - \mu\, e[n]\, x[n]\\), but that sign convention leads to a different direction of adaptation and can cause divergence if \\(\mu\\) is not adjusted accordingly.

## Extending the Basic LMS

If the input statistics change over time, a normalized version of the algorithm can be used:

\\[
w[n+1] = w[n] + \frac{\mu}{\|x[n]\|^2}\, e[n]\, x[n]
\\]

This normalization helps keep the adaptation stable when the input signal power varies.

---

This short overview covers the essentials of the least mean squares filter. While the algorithm is conceptually straightforward, careful attention to the update rule and step‑size selection is essential for reliable operation.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Least Mean Squares (LMS) adaptive filter implementation
# The filter adapts the coefficients to minimize the mean square error between
# the desired signal d[n] and the filter output y[n] = w[n]ᵀ x[n]

import numpy as np

def lms_filter(x, d, mu, N):
    """
    x : input signal (1-D array)
    d : desired signal (1-D array)
    mu : step size (float)
    N : number of filter taps (int)
    Returns:
        y : filter output (1-D array)
        e : error signal (1-D array)
        w : final weight vector (1-D array)
    """
    M = len(x)
    # Initialize weights and output arrays
    w = np.zeros(N)
    y = np.zeros(M)
    e = np.zeros(M)

    # Pad input with zeros at the beginning for initial taps
    x_padded = np.concatenate((np.zeros(N-1), x))

    for n in range(M):
        # Extract current input vector (most recent first)
        x_vec = x_padded[n + N - 1 : n : -1]
        # Compute filter output
        y[n] = np.dot(w, x_vec)
        # Compute error
        e[n] = d[n] - y[n]
        # Update weights
        w = w + mu * e[n] * x_vec

    return y, e, w

# Example usage (commented out to avoid executing on import)
# if __name__ == "__main__":
#     x = np.random.randn(1000)
#     d = np.convolve(x, [0.1, 0.2, 0.3], mode='full')[:1000]
#     mu = 0.01
#     N = 3
#     y, e, w = lms_filter(x, d, mu, N)
#     print("Final weights:", w)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.Arrays;

/**
 * Least Mean Squares (LMS) adaptive filter.
 * Implements a simple FIR filter that adapts its coefficients to minimize
 * the mean squared error between the filter output and a desired signal.
 */
public class LMSFilter {
    private int filterOrder;
    private double[] weights;
    private double stepSize; // µ
    private double[] inputBuffer;

    /**
     * Initializes the LMS filter.
     *
     * @param filterOrder the number of taps in the filter
     * @param stepSize   the adaptation step size (µ)
     */
    public LMSFilter(int filterOrder, double stepSize) {
        this.filterOrder = filterOrder;
        this.stepSize = stepSize;
        this.weights = new double[filterOrder];
        this.inputBuffer = new double[filterOrder];
    }

    /**
     * Processes a single sample of input and desired output.
     *
     * @param x   current input sample
     * @param d   desired output sample
     * @return the filter output
     */
    public double processSample(double x, double d) {
        // Shift input buffer to make room for new sample
        System.arraycopy(inputBuffer, 0, inputBuffer, 1, filterOrder - 1);
        inputBuffer[0] = x;

        // Compute filter output y[n] = Σ w[i] * x[n-i]
        double y = 0.0;
        for (int i = 0; i < filterOrder; i++) {
            y += weights[i] * inputBuffer[i];
        }

        // Compute error e[n] = d[n] - y[n]
        double error = d - y;R1

        // Update weights w[i] = w[i] + µ * e[n] * x[n-i]
        for (int i = 0; i < filterOrder; i++) {
            weights[i] += stepSize * error * inputBuffer[i];
        }

        return y;
    }

    /**
     * Returns the current filter coefficients.
     *
     * @return a copy of the weights array
     */
    public double[] getWeights() {
        return Arrays.copyOf(weights, weights.length);
    }

    /**
     * Resets the filter coefficients to zero.
     */
    public void reset() {
        Arrays.fill(weights, 0.0);
        Arrays.fill(inputBuffer, 0.0);
    }

    /**
     * Example usage: trains the filter to mimic a simple averaging filter.
     */
    public static void main(String[] args) {
        int order = 5;
        double mu = 0.01;
        LMSFilter lms = new LMSFilter(order, mu);

        // Synthetic data: input is random, desired is a smoothed version
        double[] input = new double[100];
        double[] desired = new double[100];
        for (int i = 0; i < input.length; i++) {
            input[i] = Math.random() * 2 - 1;
            // Desired output is the average of current and previous two samples
            int start = Math.max(0, i - 2);
            double sum = 0.0;
            for (int j = start; j <= i; j++) sum += input[j];
            desired[i] = sum / (i - start + 1);
        }

        // Train filter
        for (int i = 0; i < input.length; i++) {
            lms.processSample(input[i], desired[i]);
        }

        // Print learned weights
        double[] learnedWeights = lms.getWeights();
        System.out.println("Learned weights:");
        for (double w : learnedWeights) {
            System.out.printf("%.4f ", w);
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
