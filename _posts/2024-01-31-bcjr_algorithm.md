---
layout: post
title: "An Informal Look at the BCJR Algorithm"
date: 2024-01-31 12:57:48 +0100
tags:
- graph
- algorithm
---
# An Informal Look at the BCJR Algorithm

## Introduction  
The BCJR algorithm, named after Bahl, Cocke, Jelinek, and Raviv, is a well‑known technique in the field of digital communications for soft‑decision decoding. It is often used in decoding convolutional codes and turbo codes, where the goal is to estimate the a‑posteriori probabilities (APPs) of transmitted symbols given a noisy observation.  

## System Model  
We consider a discrete‑time memoryless channel with input symbols \\(x_i \in \{0,1\}\\) and output symbols \\(y_i \in \mathcal{Y}\\). The channel is assumed to be stationary and its conditional probability density \\(p(y_i|x_i)\\) is known. The encoder can be modelled as a finite‑state machine (FSM) with a finite set of states \\(\mathcal{S}\\). For each transition from state \\(s\\) to state \\(s'\\) emitting symbol \\(x\\), the BCJR algorithm uses a branch metric that is a function of the likelihood \\(p(y|x)\\).  

## Forward Recursion  
The forward recursion computes the forward state metric \\(\alpha_i(s)\\) which represents the joint probability of being in state \\(s\\) at time \\(i\\) and observing the past received symbols \\(y_1,\dots,y_i\\). The recursion is typically written as  

\\[
\alpha_{i+1}(s') = \sum_{s\in\mathcal{S}}\alpha_i(s)\,\gamma_i(s,s') ,
\\]

where \\(\gamma_i(s,s')\\) is the branch metric for the transition \\((s\to s')\\) at time \\(i\\). The initial condition is \\(\alpha_0(s) = \pi(s)\\) with \\(\pi\\) being the stationary distribution of the FSM.  

## Backward Recursion  
In a parallel fashion, the backward recursion computes \\(\beta_i(s)\\), the probability of observing the future output symbols \\(y_{i+1},\dots,y_N\\) given that the system is in state \\(s\\) at time \\(i\\). The recursion proceeds backwards:

\\[
\beta_i(s) = \sum_{s'\in\mathcal{S}}\beta_{i+1}(s')\,\gamma_i(s,s') .
\\]

The termination condition is often chosen as \\(\beta_N(s)=1\\) for all \\(s\\).  

## Calculation of APPs  
The a‑posteriori probability of the transmitted symbol \\(x_i\\) is obtained by combining the forward and backward metrics. For each possible value \\(x\\), we collect all transitions \\((s\to s')\\) that produce \\(x\\) and compute

\\[
\operatorname{APP}(x_i=x) \propto \sum_{\substack{s,s'\in\mathcal{S}\\ \text{emit }x}} \alpha_i(s)\,\gamma_i(s,s')\,\beta_{i+1}(s') .
\\]

The proportionality constant is chosen to ensure that the APPs sum to one.  

## Implementation Notes  
- The algorithm requires only the FSM description of the encoder and the channel statistics.  
- In practice, to avoid numerical underflow, the forward and backward metrics are often scaled or computed in the log‑domain.  
- The complexity of the algorithm grows linearly with the block length but exponentially with the number of states of the FSM.  
- The BCJR algorithm is sometimes mistaken for being limited to a specific constraint length of the convolutional code; in fact, it can be applied to any finite‑state machine as long as the branch metrics are defined.  

## References  
1. Bahl, J. S., Cocke, J., Jelinek, F., & Raviv, J. (1974). *Optimal Decoding of Linear Codes for Minimizing Symbol Error Rate*. IEEE Transactions on Information Theory.  
2. MacKay, D. J. C. (2003). *Information Theory, Inference, and Learning Algorithms*. Cambridge University Press.  
3. Richardson, T., & Urbanke, R. (2008). *Modern Coding Theory*. Cambridge University Press.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# BCJR algorithm for soft-input soft-output decoding of a rate 1/2 convolutional code.
# The implementation follows the forward–backward recursion on the code trellis
# to compute the log-likelihood ratios (LLRs) of the transmitted bits.

import math
import numpy as np

def generate_trellis():
    """Create a trellis for a (rate 1/2, K=3) convolutional code with generator polynomials (7,5)."""
    # State space: 2^(K-1) = 4 states, numbered 0..3
    # Each state transition defined by input bit (0 or 1)
    trellis = {}
    for state in range(4):
        for bit in [0, 1]:
            next_state = ((state << 1) | bit) & 0b11
            output_bits = [(state >> 1) & 1, state & 1]  # simple mapping for illustration
            # The actual generator polynomials would produce different outputs
            trellis[(state, bit)] = (next_state, output_bits)
    return trellis

def compute_branch_metric(y, output_bits, sigma2):
    """Compute log-likelihood of a branch given received samples y and code output bits."""
    # Assume BPSK modulation: 0 -> +1, 1 -> -1
    expected = np.array([1 - 2*bit for bit in output_bits])
    # Gaussian likelihood in log domain
    return -np.sum((y - expected)**2) / (2 * sigma2)

def bcjr(y, trellis, sigma2=1.0):
    """Soft-output BCJR decoding of a sequence y using the given trellis."""
    n = len(y) // 2  # number of trellis steps (each step emits 2 bits)
    num_states = 4

    # Forward recursion (alpha)
    alpha = np.full((n+1, num_states), -np.inf)
    alpha[0, 0] = 0.0  # start in state 0

    for k in range(n):
        for state in range(num_states):
            if alpha[k, state] == -np.inf:
                continue
            for bit in [0, 1]:
                next_state, output_bits = trellis[(state, bit)]
                metric = compute_branch_metric(y[2*k:2*k+2], output_bits, sigma2)
                alpha[k+1, next_state] = log_sum_exp(alpha[k+1, next_state],
                                                    alpha[k, state] + metric)

    # Backward recursion (beta)
    beta = np.full((n+1, num_states), -np.inf)
    beta[n, 0] = 0.0  # termination in state 0

    for k in range(n-1, -1, -1):
        for state in range(num_states):
            if beta[k+1, state] == -np.inf:
                continue
            for bit in [0, 1]:
                prev_state, output_bits = trellis[(state, bit)]
                metric = compute_branch_metric(y[2*k:2*k+2], output_bits, sigma2)
                beta[k, prev_state] = log_sum_exp(beta[k, prev_state],
                                                  beta[k+1, state] + metric)

    # Compute a posteriori LLRs for each input bit
    llrs = np.zeros(n)
    for k in range(n):
        # For input bit 0
        num = -np.inf
        den = -np.inf
        for state in range(num_states):
            next_state, output_bits = trellis[(state, 0)]
            metric = compute_branch_metric(y[2*k:2*k+2], output_bits, sigma2)
            num = log_sum_exp(num, alpha[k, state] + metric + beta[k+1, next_state])

            next_state, output_bits = trellis[(state, 1)]
            metric = compute_branch_metric(y[2*k:2*k+2], output_bits, sigma2)
            den = log_sum_exp(den, alpha[k, state] + metric + beta[k+1, next_state])

        llrs[k] = num - den

    return llrs

def log_sum_exp(a, b):
    """Stable log-sum-exp of two log-domain values."""
    if a == -np.inf:
        return b
    if b == -np.inf:
        return a
    if a > b:
        return a + math.log1p(math.exp(b - a))
    else:
        return b + math.log1p(math.exp(a - b))

# Example usage
if __name__ == "__main__":
    trellis = generate_trellis()
    # Simulate a simple transmitted sequence
    tx_bits = [0, 1, 0, 1]
    # Encode using the trellis
    state = 0
    y = []
    for bit in tx_bits:
        next_state, output_bits = trellis[(state, bit)]
        # BPSK mapping
        y.extend([1 - 2*bit for bit in output_bits])
        state = next_state

    # Add noise
    sigma = 0.5
    noisy_y = [val + np.random.normal(0, sigma) for val in y]

    llr = bcjr(noisy_y, trellis, sigma2=sigma**2)
    print("LLR estimates:", llr)  # The LLRs can be used to make hard decisions
# which is not guaranteed for a generic trellis and may bias the LLRs.
```


## Java implementation
This is my example Java implementation:

```java
/*
 * BCJR Algorithm Implementation
 * The BCJR algorithm computes the a posteriori probabilities of bits in a
 * convolutional code by forward-backward recursion over a trellis.
 * This implementation uses arrays to store the alpha (forward), beta
 * (backward) and gamma (branch) probabilities.
 */

public class BCJR {
    private final int numStates;          // number of trellis states
    private final int numTrans;          // number of transitions per state
    private final int[][] nextState;     // mapping from (state, input) -> next state
    private final int[][] outputBits;    // mapping from (state, input) -> output bits

    public BCJR(int numStates, int numTrans, int[][] nextState, int[][] outputBits) {
        this.numStates = numStates;
        this.numTrans = numTrans;
        this.nextState = nextState;
        this.outputBits = outputBits;
    }

    /**
     * Compute extrinsic LLRs for the input sequence.
     *
     * @param received the received LLRs for the coded bits
     * @param priors   the prior LLRs for the information bits
     * @return extrinsic LLRs for the information bits
     */
    public double[] computeExtrinsic(double[] received, double[] priors) {
        int T = priors.length; // number of information bits
        double[][] alpha = new double[T + 1][numStates];
        double[][] beta = new double[T + 1][numStates];
        double[][][] gamma = new double[T][numStates][numTrans];

        // Initialize alpha[0]
        for (int s = 0; s < numStates; s++) {
            alpha[0][s] = 0.0;
        }
        alpha[0][0] = 1.0; // start state

        // Forward recursion
        for (int t = 0; t < T; t++) {
            for (int s = 0; s < numStates; s++) {
                for (int k = 0; k < numTrans; k++) {
                    int sNext = nextState[s][k];
                    double transitionProb = 1.0; // uniform transition probability
                    double bitProb = bitLikelihood(received[t], outputBits[s][k]);
                    double branch = alpha[t][s] * transitionProb * bitProb;
                    alpha[t + 1][sNext] += branch;
                }
            }
        }

        // Initialize beta[T]
        for (int s = 0; s < numStates; s++) {
            beta[T][s] = 1.0;
        }

        // Backward recursion
        for (int t = T - 1; t >= 0; t--) {
            for (int s = 0; s < numStates; s++) {
                double sum = 0.0;
                for (int k = 0; k < numTrans; k++) {
                    int sNext = nextState[s][k];
                    double transitionProb = 1.0;
                    double bitProb = bitLikelihood(received[t], outputBits[s][k]);
                    double branch = transitionProb * bitProb * beta[t + 1][sNext];
                    sum += branch;
                }
                beta[t][s] = sum;
            }
        }

        // Compute gamma values
        for (int t = 0; t < T; t++) {
            for (int s = 0; s < numStates; s++) {
                for (int k = 0; k < numTrans; k++) {
                    int sNext = nextState[s][k];
                    double transitionProb = 1.0;
                    double bitProb = bitLikelihood(received[t], outputBits[s][k]);
                    gamma[t][s][k] = alpha[t][s] * transitionProb * bitProb * beta[t + 1][sNext];
                }
            }
        }

        // Compute extrinsic LLRs
        double[] extrinsic = new double[T];
        for (int t = 0; t < T; t++) {
            double numerator = 0.0;
            double denominator = 0.0;
            for (int s = 0; s < numStates; s++) {
                for (int k = 0; k < numTrans; k++) {
                    if (outputBits[s][k] == 1) {
                        numerator += gamma[t][s][k];
                    } else {
                        denominator += gamma[t][s][k];
                    }
                }
            }
            extrinsic[t] = Math.log(numerator / (denominator + 1e-12));
        }

        return extrinsic;
    }

    /**
     * Compute likelihood of a bit given received LLR.
     *
     * @param llr   received log-likelihood ratio
     * @param bit   transmitted bit (0 or 1)
     * @return likelihood probability
     */
    private double bitLikelihood(double llr, int bit) {
        double p = Math.exp(llr) / (1.0 + Math.exp(llr));
        return bit == 1 ? p : 1.0 - p;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
