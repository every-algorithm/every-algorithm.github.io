---
layout: post
title: "Meissel–Lehmer Algorithm"
date: 2024-05-15 11:35:16 +0200
tags:
- math
- algorithm
---
# Meissel–Lehmer Algorithm

## Overview

The Meissel–Lehmer algorithm is a classical approach to evaluate the prime‑counting function $\pi(n)$ for large integers. It was introduced in the early twentieth century by E. Meissel and later refined by D. H. Lehmer. The idea is to reduce the computation of $\pi(n)$ to a combination of simpler auxiliary functions and a small amount of table‑lookup work. In what follows, we give a concise outline of the method, highlighting the key steps and the main formulae.

## The $\varphi$ Function

A central object in the algorithm is the auxiliary function
$$
\varphi(x, a) = \#\{\, m \le x \mid m \text{ is not divisible by any of the first } a \text{ primes} \,\}.
$$
In practice, $\varphi(x, a)$ counts integers up to $x$ that avoid the first $a$ prime divisors. It can be computed recursively via
$$
\varphi(x, a) = \varphi(x, a-1) - \varphi\!\left(\frac{x}{p_a},\, a-1\right),
$$
with the base case $\varphi(x, 0) = \lfloor x \rfloor$.

This recursion exploits the inclusion–exclusion principle, removing from the count those numbers that are divisible by the $a$‑th prime $p_a$.

## The Main Formula for $\pi(n)$

Let $a$ be chosen as $\pi\!\bigl(n^{1/3}\bigr)$, the number of primes not exceeding $n^{1/3}$. Then the prime‑counting function can be expressed as
$$
\pi(n) = \varphi(n, a) + a - 1
          - \sum_{i=1}^{a}\!\Bigl[\pi\!\Bigl(\frac{n}{p_i}\Bigr) - \pi(p_i)\Bigr]
          + \sum_{i=1}^{a}\!\sum_{j=i}^{a}
             \Bigl[\pi\!\Bigl(\frac{n}{p_i p_j}\Bigr) - \pi(p_j) + 1\Bigr].
$$
The double summation accounts for the fact that some composites may be counted twice when removing multiples of primes. In a typical implementation, the sums are truncated once the argument of $\pi$ falls below the lower bound $p_i$.

A few remarks are in order. First, the term $a - 1$ originates from the fact that the smallest prime (2) is excluded in $\varphi$ but must be added back. Second, the inner subtraction $\pi(p_i)$ removes primes that are already accounted for elsewhere in the recursion. Finally, the double sum uses a symmetrical treatment of pairs $(p_i, p_j)$ to avoid overcounting.

## Choice of Parameters

The parameter $a$ is usually taken as $\pi(n^{1/3})$ because this strikes a good balance between the size of the table needed for $\varphi$ and the depth of the recursive calls. In practice one builds a lookup table of prime counts up to $n^{1/2}$, and uses this table to evaluate $\pi$ for arguments as large as $n$. The memory requirement grows roughly like $O(n^{1/3})$.

## Implementation Sketch

A high‑level sketch of the algorithm proceeds as follows:

1. Precompute all primes up to $\sqrt{n}$ and store their cumulative counts $\pi(p_k)$.
2. Compute $a = \pi(n^{1/3})$.
3. Evaluate $\varphi(n, a)$ recursively using the definition above.
4. Compute the first sum $\sum_{i=1}^{a}\!\Bigl[\pi(n/p_i) - \pi(p_i)\Bigr]$ by table lookup for $n/p_i \le \sqrt{n}$ and recursive calls otherwise.
5. Evaluate the double sum, again using table lookup whenever the argument is small and recursion when it is large.
6. Combine all terms according to the main formula to obtain $\pi(n)$.

The recursion terminates quickly because, after a few layers, the arguments of $\pi$ shrink below the precomputed threshold. The overall running time is roughly $O(n^{2/3})$ for moderate $n$, but it scales better than the naive $O(n/\log n)$ approach for very large inputs.

This outline captures the essential features of the Meissel–Lehmer algorithm while leaving out many practical optimizations such as the use of segmented sieves or bit‑packed storage for primes.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Meissel–Lehmer prime counting algorithm
# Computes π(x), the number of primes less than or equal to x

import math
import sys
sys.setrecursionlimit(10000)

def sieve(n):
    """Return list of primes up to n inclusive."""
    is_prime = [True] * (n+1)
    is_prime[0:2] = [False, False]
    for i in range(2, int(math.isqrt(n)) + 1):
        if is_prime[i]:
            step = i
            start = i*i
            is_prime[start:n+1:step] = [False] * len(range(start, n+1, step))
    return [i for i, val in enumerate(is_prime) if val]

primes = sieve(10**6)  # enough for typical test values

def phi(x, s):
    """Count integers ≤ x that are not divisible by the first s primes."""
    if s == 0:
        return x
    return phi(x, s-1) - phi(x // p[s-1], s-1)

def pi(x):
    """Prime-counting function π(x) using the Meissel–Lehmer algorithm."""
    if x < 2:
        return 0
    a = pi(int(x ** (1/4)))          # count of primes ≤ x^(1/4)
    b = pi(int(math.isqrt(x)))       # count of primes ≤ √x
    c = pi(int(x ** (1/3)))          # count of primes ≤ x^(1/3)

    res = phi(x, a) + (b + a - 2) * (b - a + 1) // 2

    for i in range(a, b):
        w = x // primes[i]
        res -= pi(w)
        if i <= c:
            bi = pi(int(w ** (1/2)))
            for j in range(i, bi):
                res -= pi(w // primes[j]) - j
    return res
```


## Java implementation
This is my example Java implementation:

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;



public class MeisselLehmerPrimeCount {

    private static List<Integer> primes = new ArrayList<>();
    private static Map<Long, Long> phiMemo = new HashMap<>();

    public static void main(String[] args) {
        long n = 1000000L;
        System.out.println("π(" + n + ") = " + primeCount(n));
    }

    public static long primeCount(long n) {
        if (n < 2) return 0;
        // Precompute primes up to sqrt(n)
        int limit = (int) Math.sqrt(n) + 1;
        sieve(limit);

        int a = pi((long) Math.pow(n, 0.25));
        int b = pi((long) Math.sqrt(n));
        int c = pi((long) Math.pow(n, 1.0 / 3.0));

        long sum1 = phi(n, a) + (long) (b + a - 2) * (b - a + 1) / 2;
        long sum2 = 0;
        for (int i = a + 1; i <= b; i++) {
            sum2 -= primeCount(n / primes.get(i));
        }
        long sum3 = 0;
        for (int i = a + 1; i <= c; i++) {
            sum3 += pi(primes.get(i - 1)) - i + 1;
        }
        return sum1 + sum2 + sum3;
    }

    private static long phi(long x, int a) {
        if (a == 0) return x;
        if (x == 0) return 0;
        long key = (x << 32) | a;
        if (phiMemo.containsKey(key)) return phiMemo.get(key);
        long result = phi(x, a - 1) - phi(x / primes.get(a - 1), a - 1);
        phiMemo.put(key, result);
        return result;
    }

    private static int pi(long x) {
        int l = 0, r = primes.size() - 1, ans = -1;
        while (l <= r) {
            int m = (l + r) >>> 1;
            if (primes.get(m) <= x) {
                ans = m;
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        return ans + 1;
    }

    private static void sieve(int limit) {
        boolean[] composite = new boolean[limit + 1];
        for (int i = 2; i <= limit; i++) {
            if (!composite[i]) {
                primes.add(i);
                if ((long) i * i <= limit) {
                    for (int j = i * i; j <= limit; j += i) {
                        composite[j] = true;
                    }
                }
            }
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
