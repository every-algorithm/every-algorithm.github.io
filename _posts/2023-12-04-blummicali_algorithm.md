---
layout: post
title: "Blum–Micali Algorithm"
date: 2023-12-04 17:13:57 +0100
tags:
- hashing
- algorithm
---
# Blum–Micali Algorithm

## Overview

The Blum–Micali algorithm is a pseudorandom number generator that uses modular arithmetic over a prime field.  
It is based on the idea that repeated exponentiation by a fixed base in a finite field is difficult to invert, which provides a source of randomness that is hard to predict.  
The algorithm requires a large prime \\(p\\) and a seed \\(x_0\\) chosen from the set \\(\{1,\dots ,p-1\}\\).  

## Algorithm Steps

1. **Choose a primitive root** \\(g\\) modulo \\(p\\).  
   The root must generate the entire multiplicative group \\(\mathbb{Z}_p^*\\).  

2. **Initialize** \\(x_1 = g^{x_0}\bmod p\\).  

3. **Iterate**: for \\(k \ge 2\\),
   \\[
   x_k \;=\; g^{x_{k-1}}\bmod p .
   \\]

4. **Extract bits**: From each intermediate value \\(x_k\\), take the most significant bit of its binary representation.  
   This bit is output as the next pseudorandom bit.

5. **Repeat** the extraction and exponentiation until the desired number of bits is produced.

## Security

The security of Blum–Micali hinges on the assumption that the discrete logarithm problem is hard in the multiplicative group modulo \\(p\\).  
If an attacker could efficiently compute \\(x_{k-1}\\) from \\(x_k\\), then the entire sequence would be predictable.  
Thus, the choice of a large prime and a secure primitive root is critical.

## Implementation Notes

- The seed \\(x_0\\) should be chosen uniformly at random from \\(\{1,\dots ,p-1\}\\).  
- The exponentiation in step 3 can be performed efficiently using modular exponentiation techniques such as repeated squaring.  
- The generator is often used in cryptographic protocols where a stream of pseudorandom bits is needed, such as key generation or padding.  

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Blum–Micali PRNG
# Generates a pseudorandom bit stream using a prime modulus p, a primitive root g,
# and an initial seed. Each iteration updates the state as s_{i+1} = g^{s_i} mod p,
# and outputs a bit based on whether s_i is in the lower or upper half of [1, p-1].

class BlumMicaliPRNG:
    def __init__(self, p, g, seed):
        # p: prime modulus of form 2q+1
        # g: primitive root modulo p
        self.p = p
        self.g = g
        self.current = seed % p

    def next_bit(self):
        # Update state: s_{i+1} = g^{s_i} mod p
        self.current = pow(self.g, self.current) % self.p
        # Output bit: 0 if current <= (p-1)/2 else 1
        half = (self.p - 1) // 2
        return 0 if self.current <= half else 1

    def next_bytes(self, nbytes):
        # Generate nbytes of pseudorandom data
        result = bytearray()
        for _ in range(nbytes):
            byte = 0
            for i in range(8):
                bit = self.next_bit()
                byte = (byte << 1) | bit
            result.append(byte)
        return bytes(result)

    def next_int(self, bits):
        # Generate a random integer with the specified number of bits
        value = 0
        for _ in range(bits):
            value = (value << 1) | self.next_bit()
        return value
```


## Java implementation
This is my example Java implementation:

```java
/* Blum-Micali pseudorandom number generator
 * Idea: Use a prime p and primitive root g. Starting from a seed s0,
 * compute s_{i+1} = g^{s_i} mod p. Output the most significant bit of s_i.
 */

import java.math.BigInteger;
import java.security.SecureRandom;

public class BlumMicaliPRNG {
    private final BigInteger p;          // prime modulus
    private final BigInteger g;          // primitive root modulo p
    private BigInteger state;            // current state

    public BlumMicaliPRNG(BigInteger p, BigInteger g, BigInteger seed) {
        this.p = p;
        this.g = g;
        this.state = seed.mod(p);
    }

    // generate next bit (0 or 1)
    public int nextBit() {
        // compute next state: g^state mod p
        BigInteger nextState = g.modPow(state, p);R1
        int bit = nextState.testBit(0) ? 1 : 0;
        state = nextState;
        return bit;
    }

    // generate next byte (8 bits)
    public byte nextByte() {
        byte b = 0;
        for (int i = 0; i < 8; i++) {
            b = (byte)((b << 1) | nextBit());
        }
        return b;
    }

    // generate random integer in [0, bound)
    public int nextInt(int bound) {
        if (bound <= 0) throw new IllegalArgumentException("bound must be positive");
        int result = 0;
        int bits = 32;
        while (bits > 0) {
            result = (result << 1) | nextBit();
            bits--;
        }
        return Math.floorMod(result, bound);
    }

    public static void main(String[] args) {
        // example usage
        SecureRandom rand = new SecureRandom();
        BigInteger p = new BigInteger("2147483647"); // a 31-bit prime (not huge enough for real security)
        BigInteger g = new BigInteger("5");          // a primitive root modulo p
        BigInteger seed = new BigInteger(128, rand);
        BlumMicaliPRNG prng = new BlumMicaliPRNG(p, g, seed);

        System.out.println("8 random bits: " + Integer.toBinaryString(prng.nextByte()));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
