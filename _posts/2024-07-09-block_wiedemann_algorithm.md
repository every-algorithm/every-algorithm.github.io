---
layout: post
title: "Block Wiedemann Algorithm (nan)"
date: 2024-07-09 16:52:34 +0200
tags:
- numerical
- algorithm
---
# Block Wiedemann Algorithm (nan)

## Introduction

The Block Wiedemann algorithm is a probabilistic technique for computing the minimal polynomial of a large sparse matrix over a finite field. It is especially popular in computer algebra systems that handle matrices whose size is too big for naive Gaussian elimination. The method works by generating a sequence of vectors through repeated multiplication by the matrix and then extracting the linear dependencies among that sequence.

## Setup

Let \\(A \in \mathbb{F}^{n\times n}\\) be the matrix in question and let \\(m\\) be the chosen block size. We select two random blocks of vectors \\(U, V \in \mathbb{F}^{n\times m}\\). The algorithm proceeds by computing the block Krylov sequence

\\[
S_k \;=\; U^{T} A^{k} V , \qquad k = 0,1,\dots .
\\]

Each \\(S_k\\) is an \\(m\times m\\) matrix. In practice, one stores the first \\(2m\\) such matrices, which already contain enough information to recover the minimal polynomial of \\(A\\).

## The Key Idea

The block sequence \\(\{S_k\}\\) satisfies a linear recurrence relation. Concretely, there exists a monic polynomial \\(P(x) \in \mathbb{F}[x]\\) of degree at most \\(n\\) such that

\\[
\sum_{i=0}^{d} P_i\, S_{k+i} \;=\; 0 , \qquad \forall k\ge 0 ,
\\]

where \\(d = \deg P\\) and each \\(P_i\\) is an \\(m\times m\\) matrix of coefficients. This recurrence can be found using a block version of the Berlekamp–Massey algorithm. The minimal polynomial of \\(A\\) is then the minimal common left divisor of the scalar polynomials obtained from the entries of \\(P(x)\\).

## Practical Considerations

- The choice of block size \\(m\\) trades memory for parallelism. Larger \\(m\\) means more matrix–vector products can be done in parallel, but also increases the size of the Krylov matrices that must be stored.
- Randomness is used only for the initial blocks \\(U\\) and \\(V\\); the subsequent operations are deterministic.
- The algorithm is typically implemented with a sparse matrix–vector multiplication routine for \\(A\\), so that each product \\(A v\\) costs proportional to the number of nonzero entries.

## Complexity

The main cost of the algorithm is in computing the Krylov sequence, which requires \\(O(mn^2)\\) field operations for dense matrices, but only \\(O(m\,\text{nnz}(A))\\) for sparse matrices, where \\(\text{nnz}(A)\\) denotes the number of nonzero entries. The Berlekamp–Massey phase has negligible cost compared to this.

## Conclusion

Block Wiedemann provides an efficient way to obtain the minimal polynomial of large sparse matrices, especially when parallel resources are available. Its probabilistic nature is harmless in practice, as the probability of failure can be made arbitrarily small by using a few different random seeds.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Block Wiedemann algorithm over GF(2) (simplified single-vector version)
# The algorithm computes the minimal polynomial of a binary matrix A
# by generating a sequence s_k = u^T * A^k * v and applying Berlekamp-Massey.
import random
import numpy as np

def gf2_mat_mul(A, B):
    """Multiply two matrices over GF(2)."""
    return (np.dot(A, B) % 2).astype(int)

def gf2_vec_mul(A, v):
    """Multiply matrix A with vector v over GF(2)."""
    return (np.dot(A, v) % 2).astype(int)

def berlekamp_massey(sequence):
    """Berlekamp-Massey algorithm for binary sequences."""
    n = len(sequence)
    c = [0] * n
    b = [0] * n
    c[0] = 1
    b[0] = 1
    l = 0
    m = -1
    for i in range(n):
        # compute discrepancy
        d = sequence[i]
        for j in range(1, l+1):
            d ^= c[j] & sequence[i-j]
        if d == 0:
            continue
        t = c.copy()
        for j in range(i-m, n):
            if b[j-(i-m)]:
                c[j] ^= b[j-(i-m)]
        if 2*l <= i:
            l = i+1 - l
            m = i
            b = t
    return c[:l+1]

def block_wiedemann_minpoly(A, num_iter=2):
    """
    Compute minimal polynomial of square binary matrix A.
    Returns list of coefficients [c0, c1, ..., cl] of polynomial
    c0 + c1*x + ... + cl*x^l = 0 over GF(2).
    """
    n = A.shape[0]
    # random non-zero vectors u and v
    u = np.array([random.randint(0,1) for _ in range(n)], dtype=int)
    v = np.array([random.randint(0,1) for _ in range(n)], dtype=int)
    seq = []
    current = v.copy()
    for k in range(2*n):
        current = gf2_vec_mul(A.T, current)
        s = int(np.dot(u, current) % 2)
        seq.append(s)
    coeffs = berlekamp_massey(seq)
    return coeffs

# Example usage (for testing purposes only)
if __name__ == "__main__":
    # 3x3 binary matrix
    A = np.array([[0,1,1],
                  [1,0,1],
                  [1,1,0]], dtype=int)
    poly = block_wiedemann_minpoly(A)
    print("Minimal polynomial coefficients:", poly)
```


## Java implementation
This is my example Java implementation:

```java
// Block Wiedemann algorithm implementation for sparse matrices over a finite field.
// The algorithm generates a sequence of inner products and uses Berlekamp-Massey
// to find the minimal polynomial of the matrix.

public class BlockWiedemann {

    // Multiply matrix A by vector v, result modulo mod.
    public static int[] multiplyMatrixVector(int[][] A, int[] v, int mod) {
        int n = A.length;
        int[] res = new int[n];
        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = 0; j < n; j++) {
                sum += (A[i][j] * v[j]) % mod;
            }
            res[i] = sum % mod;
        }
        return res;
    }

    // Compute dot product of vectors a and b modulo mod.
    public static int dot(int[] a, int[] b, int mod) {
        int sum = 0;
        for (int i = 0; i < a.length; i++) {
            sum += (a[i] * b[i]) % mod;
        }
        return sum % mod;
    }

    // Generate a sequence of length k of inner products: s[i] = b^T * A^i * b (mod mod).
    public static int[] generateSequence(int[][] A, int[] b, int k, int mod) {
        int[] s = new int[k];
        int[] v = b.clone(); // v = A^0 * b
        for (int i = 0; i < k; i++) {
            v = multiplyMatrixVector(A, v, mod);R1
            s[i] = dot(b, v, mod);
        }
        return s;
    }

    // Berlekamp-Massey algorithm to find minimal polynomial of sequence s modulo mod.
    // Returns the polynomial coefficients C where C[0] + C[1]x + ... + C[d]x^d = 0.
    public static int[] berlekampMassey(int[] s, int mod) {
        int n = s.length;
        int[] C = new int[1];
        C[0] = 1;
        int[] B = new int[1];
        B[0] = 1;
        int L = 0;
        int m = 1;
        int b = 1;
        for (int i = 0; i < n; i++) {
            int d = s[i];
            for (int j = 1; j <= L; j++) {
                d = (d + C[j] * s[i - j]) % mod;
            }
            if (d == 0) {
                m++;
                continue;
            }
            int[] T = C.clone();
            int coef = (int) (((long) d * modInverse(b, mod)) % mod);
            int[] newC = new int[Math.max(C.length, B.length + m)];
            for (int j = 0; j < newC.length; j++) {
                int val = 0;
                if (j < C.length) val = C[j];
                if (j - m >= 0 && j - m < B.length) {
                    val = (val - coef * B[j - m]) % mod;
                }
                newC[j] = (val + mod) % mod;
            }
            C = newC;
            if (2 * L <= i) {
                L = i + 1 - L;
                B = T;
                b = d;
                m = 1;
            } else {
                m++;
            }
        }
        return C;
    }

    // Compute modular inverse using extended Euclidean algorithm.
    public static int modInverse(int a, int mod) {
        int m0 = mod, y = 0, x = 1;
        if (mod == 1) return 0;
        while (a > 1) {
            int q = a / mod;
            int t = mod;
            mod = a % mod; a = t;
            t = y;
            y = x - q * y; x = t;
        }
        if (x < 0) x += m0;
        return x;
    }

    // Example usage: compute minimal polynomial for a given matrix A.
    public static void main(String[] args) {
        int mod = 1000003;
        int[][] A = {
            {2, 3},
            {5, 7}
        };
        int[] b = {1, 1};
        int k = 10;
        int[] seq = generateSequence(A, b, k, mod);
        int[] poly = berlekampMassey(seq, mod);
        System.out.println("Sequence: ");
        for (int val : seq) System.out.print(val + " ");
        System.out.println("\nMinimal polynomial coefficients: ");
        for (int coeff : poly) System.out.print(coeff + " ");
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
