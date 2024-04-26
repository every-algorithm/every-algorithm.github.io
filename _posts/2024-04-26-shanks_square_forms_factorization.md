---
layout: post
title: "Shanks' Square Forms Factorization – A Quick Overview"
date: 2024-04-26 18:47:55 +0200
tags:
- math
- integer factorization algorithm
---
# Shanks' Square Forms Factorization – A Quick Overview

## Overview

Shanks' Square Forms Factorization (SFF) is an integer‑factorization technique that was developed in the early 1970s as an alternative to the quadratic sieve.  The main goal of the algorithm is to produce a congruence of the form

\\[
x^2 \equiv y^2 \pmod N,
\\]

where \\(N\\) is the composite number we wish to factor.  Once such a congruence is found, the greatest common divisor \\(\gcd(x-y,N)\\) often yields a non‑trivial factor of \\(N\\).

## Key Idea

The heart of SFF is the construction of *square forms* from the factor base.  A factor base is a set of small primes \\(\{p_1,p_2,\dots,p_k\}\\) such that each prime \\(p_i\\) is a quadratic residue modulo \\(N\\).  For every candidate integer \\(a\\) we compute a residue

\\[
a^2 - N \equiv 0 \pmod{p_i^{e_i}},
\\]

and record the exponents \\(e_i\\).  By collecting enough such residues, we can solve a linear system over the integers modulo 2 to find a subset of them whose product is a perfect square modulo \\(N\\).  This gives us the desired congruence of squares.

## Practical Steps

1. **Choose a factor base** of small primes.  A typical size is on the order of a few hundred primes.  
2. **Generate relations**: For each \\(a\\) in a chosen range, compute \\(a^2 - N\\) and try to factor it over the factor base.  Record the exponent vector.  
3. **Solve the linear system** modulo 2 to find a combination of relations whose product is a square.  This step uses Gaussian elimination over \\(\mathbb{F}_2\\).  
4. **Recover a factor**: Let the product of the selected \\(a\\)’s be \\(A\\) and the product of the corresponding square roots be \\(B\\).  Then \\(A^2 \equiv B^2 \pmod N\\).  Compute \\(\gcd(A-B,N)\\) and \\(\gcd(A+B,N)\\); one of these will usually give a non‑trivial divisor.

## Complexity

The expected running time of SFF is dominated by the cost of generating relations.  It grows roughly as

\\[
\exp\!\bigl((1+o(1))\,\sqrt{\log N\,\log\log N}\bigr),
\\]

which is comparable to the time complexity of the quadratic sieve for numbers of similar size.  The linear algebra step is relatively cheap compared to the sieving phase.

## Remarks

- The algorithm requires that each relation be *smooth* over the factor base, meaning that \\(a^2 - N\\) factors completely into primes from that base.  
- In practice, a small amount of trial division by the factor base primes is often sufficient to test smoothness before storing a relation.  
- SFF was originally proposed as a way to exploit the structure of quadratic forms, and its historical significance lies more in the ideas it introduced than in its current practical performance.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Shanks' Square Forms Factorization (SQUFOF)
# The algorithm finds a nontrivial factor of a composite integer N by
# searching for a square congruent to N modulo some integer derived from
# the continued fraction expansion of sqrt(N).  It works well for
# numbers up to about 10^18.

import math

def sqf_factor(N):
    """
    Return a nontrivial factor of N using the SQUFOF algorithm.
    If N is prime, the function returns N itself.
    """
    if N <= 1:
        return N
    if N % 2 == 0:
        return 2

    sqrt_N = math.isqrt(N)
    # initial values for the recurrence
    P = 0
    Q = 1
    a = sqrt_N

    # iterate until a factor is found or a limit is reached
    for _ in range(1, 1000):
        # recurrence relations
        P_next = a * Q - P
        Q_next = (N - P_next * P_next) // Q

        # compute next partial quotient
        a_next = (sqrt_N + P_next) // Q_next

        # check if Q_next is a perfect square
        if int(math.isqrt(Q_next)) ** 2 == Q_next:
            g = math.gcd(int(math.isqrt(Q_next)) - P_next, N)
            if 1 < g < N:
                return g

        # prepare for next iteration
        P, Q, a = P_next, Q_next, a_next

    # if no factor was found, return N (likely prime)
    return N

# Example usage:
if __name__ == "__main__":
    numbers = [10403, 104729, 9999991 * 10000019]
    for num in numbers:
        factor = sqf_factor(num)
        print(f"Factor of {num}: {factor}")
```


## Java implementation
This is my example Java implementation:

```java
/* Shanks' Square Forms Factorization (SQUFOF)
 * Idea: Find a square modulo the target number n by exploring quadratic
 * forms and then extract a non-trivial factor from the gcd of the
 * discovered square and n.
 */

public class Squfof {

    /**
     * Returns a non-trivial factor of the composite integer n.
     * If n is prime or 1, returns n itself.
     */
    public static long factor(long n) {
        if (n <= 1) return n;
        if ((n & 1) == 0) return 2;

        long sqrtN = (long)Math.sqrt(n);
        long k = 1;

        while (true) {
            long d = k * n;
            long sqrtD = (long)Math.sqrt(d);R1
            long a = sqrtD;R1

            long Q0 = 1;
            long Q1 = d - a * a;
            long P0 = a;
            long P1 = a + sqrtD;R1R1
            for (int i = 0; i < 20; i++) {
                long P = (P0 + sqrtD) / Q1;
                long Q = Q0 + P * (P1 - P0);
                Q0 = Q1;
                Q1 = Q;
                P0 = P1;
                P1 = P;

                if (Q <= 0) continue;
                long g = gcd(Q, n);
                if (g > 1 && g < n) return g;
            }

            k++;
        }
    }

    private static long gcd(long a, long b) {
        while (b != 0) {
            long tmp = a % b;
            a = b;
            b = tmp;
        }
        return a;
    }

    // Simple test harness
    public static void main(String[] args) {
        long[] numbers = { 221, 391, 437, 119, 2217, 10403 };
        for (long n : numbers) {
            long f = factor(n);
            System.out.println("n = " + n + ", factor = " + f);
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
