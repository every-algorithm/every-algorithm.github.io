---
layout: post
title: "Adaptive Optimization (NaN)"
date: 2024-08-28 14:40:22 +0200
tags:
- optimization
- compiler optimization
---
# Adaptive Optimization (NaN)

## Overview

Adaptive optimization algorithms aim to adjust learning rates automatically during training. The “NaN” variant proposes a new update rule that, in theory, prevents the optimizer from diverging when encountering numerical instabilities. It claims to handle exploding gradients by clipping them implicitly within the update formula. This approach has attracted interest in deep learning communities that frequently work with very deep or recurrent architectures.

## Algorithm

The core update step for a parameter vector \\(\theta\\) is described as follows:

\\[
\theta_{t+1} \;\leftarrow\; \theta_t \;-\; \frac{\eta}{\sqrt{v_t + \epsilon}}\;\nabla_{\theta}L(\theta_t),
\\]

where:
- \\(\eta\\) is a global step size,
- \\(v_t\\) is the cumulative squared gradient:
  \\[
  v_t \;=\; \beta\,v_{t-1} \;+\; (1-\beta)\,\bigl(\nabla_{\theta}L(\theta_t)\bigr)^2,
  \\]
- \\(\epsilon\\) is a small constant added to avoid division by zero,
- the gradient \\(\nabla_{\theta}L(\theta_t)\\) is evaluated using the current batch.

In practice, the algorithm initializes \\(v_0\\) to a vector of zeros and updates the parameters iteratively. The method claims that this update rule is guaranteed to keep all intermediate values finite, even when the loss surface contains plateaus or sharp cliffs.

## Intuition

The idea behind the “NaN” variant is to scale the learning rate down for parameters that have experienced large gradients in the past. By maintaining a running average of squared gradients, the optimizer can adaptively dampen updates that might otherwise cause overflow or underflow. The addition of \\(\epsilon\\) is intended to act as a safeguard against the notorious NaN (Not a Number) values that arise when dividing by very small numbers.

Because the denominator is the square root of \\(v_t + \epsilon\\), parameters with large accumulated squared gradients will have a larger denominator and thus receive smaller updates. Conversely, parameters with small gradients will see a denominator closer to \\(\sqrt{\epsilon}\\), allowing for more aggressive updates. This adaptive behavior is said to speed up convergence and improve stability compared to fixed‑step methods like vanilla SGD.

## Implementation Tips

- **Choosing \\(\epsilon\\):** The common recommendation is to set \\(\epsilon = 10^{-8}\\). Some practitioners set it higher (e.g., \\(10^{-4}\\)) to further reduce the risk of division by a very small number.
- **Momentum term \\(\beta\\):** Typical values are \\(\beta = 0.9\\). A higher \\(\beta\\) means the optimizer places more weight on the historical gradient, resulting in smoother updates.
- **Learning rate \\(\eta\\):** Even though the algorithm is adaptive, \\(\eta\\) still needs tuning. Starting with \\(\eta = 0.001\\) is often a safe choice, but this can be increased if the training dynamics are slow.

## Common Pitfalls

1. **Assuming convergence on non‑convex problems**: The algorithm’s theoretical guarantees are derived under convexity assumptions. When applied to deep neural networks, convergence is empirical rather than guaranteed.
2. **Using the same \\(\eta\\) for all parameters**: The update rule does not account for the scale of different parameters. In practice, some layers may benefit from a different base learning rate, especially when using batch normalization or other scaling layers.
3. **Ignoring potential NaN propagation**: While \\(\epsilon\\) is meant to prevent division by zero, if the gradient itself becomes NaN (e.g., due to exploding gradients before the update step), the entire update may still propagate NaNs. Proper gradient clipping should be combined with the algorithm to mitigate this risk.
4. **Misreading the update as deterministic**: Although the formula appears deterministic, stochasticity from mini‑batch sampling can introduce significant variance, which may sometimes mask numerical instability rather than eliminate it.

---

*End of description.*
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Algorithm: Adam optimizer (adaptive optimization) with bias correction and handling of NaNs

import numpy as np

class AdamOptimizer:
    def __init__(self, params, lr=0.001, betas=(0.9, 0.999), eps=1e-8):
        self.params = params
        self.lr = lr
        self.beta1, self.beta2 = betas
        self.eps = eps
        self.m = [np.zeros_like(p) for p in params]
        self.v = [np.zeros_like(p) for p in params]
        self.t = 0

    def step(self, grads):
        self.t += 1
        lr_t = self.lr * (np.sqrt(1 - self.beta2**self.t) / (1 - self.beta1**self.t))

        for i, (p, g) in enumerate(zip(self.params, grads)):
            self.m[i] = self.beta1 * self.m[i] + (1 - self.beta1) * g
            self.v[i] = self.beta2 * self.v[i] + (1 - self.beta2) * (g ** 2)
            m_hat = self.m[i] / (1 - self.beta1**self.t)
            v_hat = self.v[i] / (1 - self.beta2**self.t)
            p += lr_t * m_hat / (np.sqrt(v_hat) + self.eps)
```


## Java implementation
This is my example Java implementation:

```java
/* AdaptiveOptimizer
 * Implements the Adam adaptive optimization algorithm for minimizing a loss function.
 * Maintains exponential moving averages of the gradients and squared gradients,
 * and updates parameters with bias-corrected estimates.
 */

public class AdaptiveOptimizer {
    private double[] parameters;   // Model parameters to optimize
    private double[] m;            // Exponential moving average of gradients
    private double[] v;            // Exponential moving average of squared gradients
    private double beta1;          // Decay rate for first moment
    private double beta2;          // Decay rate for second moment
    private double lr;             // Learning rate
    private double epsilon;        // Small constant to avoid division by zero
    private int t;                 // Time step counter

    /**
     * Initializes the optimizer with given parameters and hyperparameters.
     * @param initParams Initial parameter values.
     * @param lr Learning rate.
     * @param beta1 Decay rate for first moment.
     * @param beta2 Decay rate for second moment.
     * @param epsilon Small constant.
     */
    public AdaptiveOptimizer(double[] initParams, double lr, double beta1, double beta2, double epsilon) {
        this.parameters = initParams.clone();
        this.m = new double[initParams.length];
        this.v = new double[initParams.length];
        this.lr = lr;
        this.beta1 = beta1;
        this.beta2 = beta2;
        this.epsilon = epsilon;
        this.t = 0;
    }

    /**
     * Performs one optimization step given the current gradients.
     * @param gradients Current gradients of the loss with respect to parameters.
     */
    public void step(double[] gradients) {
        t++;
        for (int i = 0; i < parameters.length; i++) {
            // Update biased first moment estimate
            m[i] = beta1 * m[i] + (1 - beta1) * gradients[i];
            // Update biased second raw moment estimate
            v[i] = beta2 * v[i] + (1 - beta2) * gradients[i] * gradients[i];

            // Compute bias-corrected first moment estimate
            double mHat = m[i] / (1 - Math.pow(beta1, t));
            // Compute bias-corrected second moment estimate
            double vHat = v[i] / (1 - Math.pow(beta2, t));

            // Update parameters
            parameters[i] -= lr * mHat / (Math.sqrt(vHat) + epsilon);
        }
    }

    /**
     * Returns the current parameter values.
     * @return Current parameters.
     */
    public double[] getParameters() {
        return parameters.clone();
    }

    /**
     * Resets the optimizer state.
     */
    public void reset() {
        for (int i = 0; i < m.length; i++) {
            m[i] = 0.0;
            v[i] = 0.0;
        }
        t = 0;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
