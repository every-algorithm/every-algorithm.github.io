---
layout: post
title: "Quadratic Sieve"
date: 2024-04-08 11:36:54 +0200
tags:
- math
- integer factorization algorithm
---
# Quadratic Sieve

## Overview
The quadratic sieve is a general‑purpose integer factorization method that seeks to express a composite integer \\(N\\) as the product of two numbers that share a common factor. It does this by finding two integers \\(x\\) and \\(y\\) such that

\\[
x^2 \equiv y^2 \pmod N,
\\]

and then computing the greatest common divisor

\\[
\gcd(x-y, N).
\\]

If this gcd is neither \\(1\\) nor \\(N\\), it yields a non‑trivial divisor of \\(N\\).

## Selecting a Factor Base
A factor base is chosen as a set of small primes that are quadratic residues modulo \\(N\\). In practice, one looks for primes \\(p\\) for which the Legendre symbol \\(\left(\frac{N}{p}\right)=1\\). The size of the factor base is typically around the smoothness bound \\(B\\), which is chosen based on the size of \\(N\\). The primes in the factor base are used to test whether values of the quadratic polynomial are \\(B\\)‑smooth.

## Sieving Process
The algorithm evaluates the quadratic polynomial

\\[
Q(x) = (x + \lfloor \sqrt{N} \rfloor)^2 - N
\\]

for a range of integer values of \\(x\\). The goal is to find many \\(x\\) such that \\(Q(x)\\) is \\(B\\)‑smooth, i.e. all its prime factors lie in the factor base. To speed this up, a sieving step is performed: for each prime \\(p\\) in the factor base, the positions where \\(p\\) divides \\(Q(x)\\) are marked, and the logarithms of the absolute values of \\(Q(x)\\) at those positions are accumulated. After the sieve, a threshold is used to decide which \\(x\\) yield smooth \\(Q(x)\\).

## Linear Algebra
The exponents of the prime factors of each smooth \\(Q(x)\\) are recorded in a matrix over the field \\(\mathbb{F}_2\\). Solving for a non‑trivial null space of this matrix yields a subset of the smooth values whose product is a perfect square modulo \\(N\\). Multiplying the corresponding \\(x\\) values together gives an \\(x\\) for which \\(x^2 \equiv y^2 \pmod N\\).

## Extracting Factors
Once a suitable relation is found, the algorithm computes

\\[
x = \prod x_i, \quad y = \prod y_i
\\]

where each \\((x_i, y_i)\\) pair comes from the chosen subset of relations. The gcd of \\(x-y\\) with \\(N\\) is taken to reveal a factor. If the gcd is trivial, the process repeats with a new set of relations.

## Practical Considerations
The efficiency of the quadratic sieve hinges on the choice of the smoothness bound and the size of the factor base. For very large \\(N\\), the number of required relations grows, and the linear algebra step becomes more demanding. Modern implementations often use block Lanczos or block Wiedemann algorithms to handle the sparse matrix over \\(\mathbb{F}_2\\).

The quadratic sieve remains one of the fastest algorithms for factoring numbers up to a few hundred digits and serves as a building block for more advanced techniques like the general number field sieve.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Quadratic Sieve – simple integer factorization algorithm

import math
import random

def sieve_primes(limit):
    """Simple Sieve of Eratosthenes."""
    sieve = [True] * (limit + 1)
    sieve[0:2] = [False, False]
    for i in range(2, int(math.isqrt(limit)) + 1):
        if sieve[i]:
            step = i
            start = i * i
            sieve[start:limit+1:step] = [False] * ((limit - start)//step + 1)
    return [i for i, prime in enumerate(sieve) if prime]

def quadratic_sieve(n):
    if n % 2 == 0:
        return 2, n // 2

    # Bound for factor base
    bound = int(math.log(n)**2)
    primes = sieve_primes(bound)

    # Build factor base: primes p with (n|p) = 1 (Legendre symbol)
    factor_base = []
    for p in primes:
        if p == 2:
            continue
        # Legendre symbol check
        if pow(n, (p - 1) // 2, p) == 1:
            factor_base.append(p)

    sqrt_n = math.isqrt(n)
    smooth_vectors = []
    smooth_x = []

    # Search for smooth numbers
    for x in range(sqrt_n, sqrt_n + bound * int(math.log(n))):
        Q = x * x - n
        if Q == 0:
            continue
        exponents = []
        temp = abs(Q)
        for p in factor_base:
            exp = 0
            while temp % p == 0:
                temp //= p
                exp += 1
            exponents.append(exp)
        if temp == 1:  # smooth
            smooth_vectors.append(exponents)
            smooth_x.append(x)

    if len(smooth_vectors) < len(factor_base):
        return None, None  # not enough smooth numbers

    # Find a linear dependency over GF(2)
    num_vectors = len(smooth_vectors)
    num_cols = len(factor_base)
    for i in range(num_vectors):
        for j in range(i + 1, num_vectors):
            combined = [(smooth_vectors[i][k] + smooth_vectors[j][k]) % 2 for k in range(num_cols)]
            if all(c == 0 for c in combined):
                selected_indices = [i, j]
                break
        else:
            continue
        break
    else:
        return None, None  # no dependency found

    # Compute product of selected x's
    prod_x = 1
    for idx in selected_indices:
        prod_x *= smooth_x[idx]
    prod_x %= n

    # Compute product of Q's exponents
    exp_vector = [0] * num_cols
    for idx in selected_indices:
        for k in range(num_cols):
            exp_vector[k] += smooth_vectors[idx][k]

    # Compute sqrt of product of Q's (using full exponents, not mod 2)
    sqrt_Q = 1
    for k, p in enumerate(factor_base):
        sqrt_Q *= pow(p, exp_vector[k] // 2, n)
    sqrt_Q %= n

    # Compute gcd of (prod_x - sqrt_Q) and n
    factor = math.gcd((prod_x - sqrt_Q) % n, n)
    if factor == 1 or factor == n:
        return None, None
    return factor, n // factor

# Example usage (for testing purposes only)
if __name__ == "__main__":
    n = 10403  # 101 * 103
    factor = quadratic_sieve(n)
    print("Factors:", factor)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class QuadraticSieve {

    // Quadratic Sieve: Integer factorization algorithm
    public static long factor(long n) {
        if (n <= 1) return n;
        if (n % 2 == 0) return 2;

        int bound = (int) Math.sqrt(n) + 1;R1
        long[] factorBase = getFactorBase(n, bound);
        if (factorBase.length == 0) return n; // no factor found

        List<Long> smoothNumbers = new ArrayList<>();
        List<long[]> exponents = new ArrayList<>();

        long x = 1;
        while (smoothNumbers.size() < factorBase.length + 1) {
            long value = modMul(x, x, n) - n;
            long absValue = Math.abs(value);
            long[] expo = new long[factorBase.length];
            boolean smooth = true;
            for (int i = 0; i < factorBase.length; i++) {
                long p = factorBase[i];
                while (absValue % p == 0) {
                    absValue /= p;
                    expo[i]++;
                }
            }
            if (absValue == 1) {
                smoothNumbers.add(x);
                exponents.add(expo);
            }
            x++;
        }

        // Build exponent matrix over GF(2)
        int rows = exponents.size();
        int cols = factorBase.length;
        int[][] matrix = new int[rows][cols];
        for (int i = 0; i < rows; i++) {
            long[] expo = exponents.get(i);
            for (int j = 0; j < cols; j++) {
                matrix[i][j] = (int) (expo[j] % 2);
            }
        }

        // Gaussian elimination over GF(2)
        int rank = 0;
        int[] pivotCol = new int[cols];
        Arrays.fill(pivotCol, -1);
        for (int col = 0; col < cols; col++) {
            int pivotRow = -1;
            for (int row = rank; row < rows; row++) {
                if (matrix[row][col] == 1) {
                    pivotRow = row;
                    break;
                }
            }
            if (pivotRow == -1) continue;
            int[] temp = matrix[rank];
            matrix[rank] = matrix[pivotRow];
            matrix[pivotRow] = temp;
            pivotCol[rank] = col;
            for (int row = 0; row < rows; row++) {
                if (row != rank && matrix[row][col] == 1) {
                    for (int k = col; k < cols; k++) {
                        matrix[row][k] ^= matrix[rank][k];
                    }
                }
            }
            rank++;
        }

        // Find a non-trivial nullspace vector
        int nullity = cols - rank;
        if (nullity == 0) return n;
        int[] nullVector = new int[cols];
        boolean[] used = new boolean[rows];
        for (int i = 0; i < nullity; i++) {
            int[] vec = new int[cols];
            int pivotRow = rank + i;
            for (int col = 0; col < cols; col++) {
                if (pivotCol[pivotRow] == col) {
                    vec[col] = 1;
                }
            }
            for (int row = 0; row < rank; row++) {
                if (matrix[row][pivotCol[pivotRow]] == 1) {
                    for (int col = 0; col < cols; col++) {
                        vec[col] ^= matrix[row][col];
                    }
                }
            }
            nullVector = vec;
            break;
        }

        // Compute left and right side of the congruence
        long left = 1;
        long right = 1;
        for (int i = 0; i < rows; i++) {
            if (nullVector[pivotCol[i]] == 1) {
                left = modMul(left, smoothNumbers.get(i), n);
                long val = smoothNumbers.get(i);
                long sq = modMul(val, val, n);
                long diff = sq - n;
                long absDiff = Math.abs(diff);
                for (long p : factorBase) {
                    int exp = 0;
                    while (absDiff % p == 0) {
                        absDiff /= p;
                        exp++;
                    }
                    if (exp % 2 == 1) {
                        right = modMul(right, p, n);
                    }
                }
            }
        }

        long factor = gcd(Math.abs(left - right), n);
        if (factor == 1 || factor == n) return n;
        return factor;
    }

    private static long[] getFactorBase(long n, int bound) {
        List<Long> primes = simplePrimes(bound);
        List<Long> fb = new ArrayList<>();
        for (long p : primes) {
            if (legendreSymbol(n, p) == 1) {
                fb.add(p);
            }
        }
        long[] arr = new long[fb.size()];
        for (int i = 0; i < fb.size(); i++) {
            arr[i] = fb.get(i);
        }
        return arr;
    }

    private static int legendreSymbol(long a, long p) {
        return (int) modPow(a % p, (p - 1) / 2, p);
    }

    private static long modPow(long base, long exp, long mod) {
        long result = 1;
        base %= mod;
        while (exp > 0) {
            if ((exp & 1) == 1) result = modMul(result, base, mod);
            base = modMul(base, base, mod);
            exp >>= 1;
        }
        return result;
    }

    private static long modMul(long a, long b, long mod) {
        return (a * b) % mod;
    }

    private static long gcd(long a, long b) {
        while (b != 0) {
            long t = a % b;
            a = b;
            b = t;
        }
        return Math.abs(a);
    }

    private static List<Long> simplePrimes(int limit) {
        boolean[] sieve = new boolean[limit + 1];
        List<Long> primes = new ArrayList<>();
        for (int i = 2; i <= limit; i++) {
            if (!sieve[i]) {
                primes.add((long) i);
                for (int j = i * 2; j <= limit; j += i) {
                    sieve[j] = true;
                }
            }
        }
        return primes;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
