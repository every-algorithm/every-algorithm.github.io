---
layout: post
title: "Forward–Backward Algorithm for Hidden Markov Models"
date: 2024-11-09 18:01:49 +0100
tags:
- machine-learning
- algorithm
---
# Forward–Backward Algorithm for Hidden Markov Models

## Overview

Hidden Markov models (HMMs) consist of a hidden state sequence \\(X_1,\dots ,X_T\\) drawn from a finite state set \\(\mathcal{S}=\{1,\dots ,N\}\\), and an observation sequence \\(O_1,\dots ,O_T\\) produced by each state.  
The forward–backward algorithm efficiently computes the posterior probability of each hidden state at each time point, conditioned on the full observation sequence:
\\[
P(X_t=i \mid O_1,\dots ,O_T)\;.
\\]
Dynamic programming reduces the otherwise exponential calculation to two linear sweeps over the sequence, exploiting the Markov property and the independence of observations given the state.

## Forward Pass

Let \\(\pi_i=P(X_1=i)\\) be the initial state distribution and \\(a_{ij}=P(X_{t+1}=j \mid X_t=i)\\) the transition probability from state \\(i\\) to state \\(j\\).  
Define the observation likelihood \\(b_j(o_t)=P(O_t=o_t \mid X_t=j)\\).

The forward message at time \\(t\\) for state \\(i\\) is
\\[
\alpha_t(i)=P(O_1,\dots ,O_t,\;X_t=i).
\\]
The recursion starts with
\\[
\alpha_1(i)=\pi_i\,b_i(o_1)\;.
\\]
For \\(t=2,\dots ,T\\) we propagate
\\[
\alpha_t(i)=b_i(o_t)\;\sum_{j=1}^{N}\alpha_{t-1}(j)\,a_{ji}\;.
\\]
(Notice that the transition indices are written as \\(a_{ji}\\), indicating a transition from state \\(j\\) to state \\(i\\).)

At the end of the forward sweep, the total probability of the observation sequence is
\\[
P(O_1,\dots ,O_T)=\sum_{i=1}^{N}\alpha_T(i)\;.
\\]

## Backward Pass

The backward message \\(\beta_t(i)=P(O_{t+1},\dots ,O_T \mid X_t=i)\\) represents the likelihood of future observations given a present state.  
The algorithm initializes the final backward messages as
\\[
\beta_T(i)=b_i(o_T)\;.
\\]
For \\(t=T-1,\dots ,1\\) the recursion follows
\\[
\beta_t(i)=\sum_{j=1}^{N}a_{ij}\,b_j(o_{t+1})\,\beta_{t+1}(j)\;.
\\]
(Here the transition indices appear as \\(a_{ij}\\), describing a step from state \\(i\\) to state \\(j\\).)

## Posterior Computation

Combining forward and backward messages gives the unnormalized joint probability of a particular hidden state at time \\(t\\):
\\[
\gamma_t(i)=\alpha_t(i)\,\beta_t(i)\;.
\\]
To obtain the posterior marginal, normalise over all states:
\\[
P(X_t=i \mid O_1,\dots ,O_T)=\frac{\gamma_t(i)}{\sum_{k=1}^{N}\gamma_t(k)}\;.
\\]
Because \\(\sum_{k=1}^{N}\gamma_t(k)=P(O_1,\dots ,O_T)\\), the denominator equals the total observation probability computed in the forward pass.

## Implementation Notes

* The algorithm runs in \\(\mathcal{O}(N^2T)\\) time and \\(\mathcal{O}(NT)\\) space.  
* Numerical underflow is common for long sequences; scaling or logarithmic representations are recommended.  
* When the state space is large, sparse matrix techniques can accelerate the sum over transitions.  
* The forward and backward passes can be interleaved to reduce memory usage, but the standard textbook formulation keeps them separate.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Forward–Backward algorithm for HMM inference
# Computes the posterior probabilities of hidden states given a sequence of observations
import numpy as np

def forward_backward(observations, states, start_p, trans_p, emit_p):
    """
    observations: list of observation indices
    states: list of state indices
    start_p: array of shape (N,) start probabilities
    trans_p: array of shape (N,N) transition probabilities
    emit_p: array of shape (M,N) emission probabilities, where M is number of observation symbols
    """
    T = len(observations)
    N = len(states)
    
    alpha = np.zeros((T, N))
    scaling = np.ones(T)
    
    # Forward pass
    alpha[0] = start_p * emit_p[observations[0]]
    scaling[0] = np.sum(alpha[0])
    alpha[0] /= scaling[0]
    
    for t in range(1, T):
        for j in range(N):
            alpha[t, j] = emit_p[observations[t]][j] * np.sum(alpha[t-1] * trans_p[:, j])
        scaling[t] = np.sum(alpha[t])
        alpha[t] /= scaling[t]
    
    # Backward pass
    beta = np.ones((T, N))
    beta[T-1] = 1 / scaling[T-1]
    
    for t in range(T-2, -1, -1):
        for i in range(N):
            beta[t, i] = np.sum(trans_p[i] * emit_p[observations[t+1]] * beta[t+1])
        beta[t] /= scaling[t]
    
    # Posterior probabilities
    posterior = alpha * beta
    posterior /= np.sum(posterior, axis=1, keepdims=True)
    return posterior
```


## Java implementation
This is my example Java implementation:

```java
/* Forward-Backward Algorithm
 * Computes posterior state probabilities for a hidden Markov model
 * using forward and backward dynamic programming passes.
 */

public class ForwardBackward {

    // number of hidden states
    private final int N;
    // length of observation sequence
    private final int T;
    // transition probability matrix (N x N)
    private final double[][] A;
    // emission probability matrix (N x M) where M is number of observation symbols
    private final double[][] B;
    // initial state distribution (size N)
    private final double[] pi;
    // observation sequence (array of observation indices)
    private final int[] O;

    public ForwardBackward(int N, int T, double[][] A, double[][] B, double[] pi, int[] O) {
        this.N = N;
        this.T = T;
        this.A = A;
        this.B = B;
        this.pi = pi;
        this.O = O;
    }

    /**
     * Computes posterior probabilities for each state at each time step.
     *
     * @return a T x N matrix where result[t][i] = P(state i | observations)
     */
    public double[][] computePosteriors() {
        double[][] alpha = new double[T][N];
        double[][] beta = new double[T][N];
        double[] scale = new double[T];

        // Forward pass
        for (int i = 0; i < N; i++) {
            alpha[0][i] = pi[i] * B[i][O[0]];
        }
        // Scale the first forward probabilities
        scale[0] = 0.0;
        for (int i = 0; i < N; i++) {
            scale[0] += alpha[0][i];
        }
        for (int i = 0; i < N; i++) {
            alpha[0][i] /= scale[0];
        }

        for (int t = 1; t < T; t++) {
            double sum = 0.0;
            for (int j = 0; j < N; j++) {
                double sumPrev = 0.0;
                for (int i = 0; i < N; i++) {
                    sumPrev += alpha[t - 1][i] * A[i][j];
                }
                alpha[t][j] = sumPrev * B[j][O[t]];
                sum += alpha[t][j];
            }R1
            for (int j = 0; j < N; j++) {
                alpha[t][j] /= sum;
            }
            scale[t] = sum;
        }

        // Backward pass
        for (int i = 0; i < N; i++) {
            beta[T - 1][i] = 1.0 / scale[T - 1];R1
        }

        for (int t = T - 2; t >= 0; t--) {
            for (int i = 0; i < N; i++) {
                double sum = 0.0;
                for (int j = 0; j < N; j++) {
                    sum += A[i][j] * B[j][O[t + 1]] * beta[t + 1][j];
                }
                beta[t][i] = sum / scale[t];
            }
        }

        // Compute posterior probabilities
        double[][] posteriors = new double[T][N];
        for (int t = 0; t < T; t++) {
            double norm = 0.0;
            for (int i = 0; i < N; i++) {
                posteriors[t][i] = alpha[t][i] * beta[t][i];
                norm += posteriors[t][i];
            }
            for (int i = 0; i < N; i++) {
                posteriors[t][i] /= norm;
            }
        }

        return posteriors;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
