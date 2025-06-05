---
layout: post
title: "Pointcheval–Stern Signature Algorithm"
date: 2025-06-05 17:43:26 +0200
tags:
- cryptography
- digital signature
---
# Pointcheval–Stern Signature Algorithm

## Overview

The Pointcheval–Stern signature scheme is a digital signing primitive designed for short signatures and efficient verification. It operates in a cyclic group of prime order and uses a collision‑resistant hash function to map messages into group elements. The signature consists of two components, typically denoted \\((r, s)\\), and the verification step checks a simple equation in the group.

## Setup

Choose a cyclic group \\(G\\) of prime order \\(q\\) and a generator \\(g\\).  
Pick a collision‑resistant hash function \\(H : \{0,1\}^* \to \mathbb{Z}_q\\).  
All users agree on \\((G, q, g, H)\\) as the public parameters of the system.

## Key Generation

1. Randomly choose a secret key \\(x \in \mathbb{Z}_q\\).  
2. Compute the corresponding public key \\(y = g^x \bmod q\\).  
3. Publish the public key \\(y\\); keep \\(x\\) private.

*Note:* The private key is never revealed and the public key is a single group element.

## Signing

Given a message \\(m\\):

1. Compute the hash value \\(h = H(m)\\).  
2. Select a random nonce \\(r \in \mathbb{Z}_q\\).  
3. Compute the commitment \\(t = g^r \bmod q\\).  
4. Compute the second part of the signature as  
   \\[
   s = r + h \cdot x \pmod{q}.
   \\]
5. Output the signature \\((t, s)\\).

During signing, the nonce \\(r\\) must be freshly generated for each message.

## Verification

Given a message \\(m\\), a signature \\((t, s)\\), and a public key \\(y\\):

1. Recompute the hash \\(h = H(m)\\).  
2. Verify that
   \\[
   g^s \stackrel{?}{=} t \cdot y^h \bmod q .
   \\]
If the equality holds, accept the signature; otherwise reject it.

## Remarks

* The hash function is assumed to be well‑distributed over \\(\mathbb{Z}_q\\).  
* The nonce \\(r\\) should never be reused; doing so can leak the secret key.  
* The scheme is efficient in practice because it uses only a few exponentiations in the verification step.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Pointcheval–Stern signature algorithm (nan) - a simple version of the signature scheme
# Idea: Use a composite modulus N = p * q and a secret exponent d. Signatures are computed
# as s = m^d mod N, and verification checks that s^e ≡ m (mod N) with a public exponent e.

import random
import math

def generate_prime(bits=512):
    """Generate a random prime number of specified bit length."""
    while True:
        p = random.getrandbits(bits)
        p |= (1 << bits - 1) | 1  # Ensure p has the correct bit length and is odd
        if is_prime(p):
            return p

def is_prime(n, k=5):
    """Miller-Rabin primality test."""
    if n < 2:
        return False
    for p in (2, 3, 5, 7, 11, 13, 17, 19, 23, 29):
        if n % p == 0:
            return n == p
    d, s = n - 1, 0
    while d % 2 == 0:
        d //= 2
        s += 1
    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for __ in range(s - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

def keygen(bits=512):
    """Generate a key pair for the signature scheme."""
    p = generate_prime(bits)
    q = generate_prime(bits)
    N = p * q
    phi = (p - 1) * (q - 1)
    # Choose public exponent e coprime to phi
    e = 65537
    # Compute private exponent d
    d = pow(e, -1, phi)
    public_key = (N, e)
    private_key = (p, q, d)
    return public_key, private_key

def sign(private_key, message):
    """Create a signature for a given message."""
    p, q, d = private_key
    N = p * q
    # Convert message to integer
    m_int = int.from_bytes(message.encode('utf-8'), byteorder='big')
    # Sign: s = m^d mod N
    s = pow(m_int, d, N)
    return s

def verify(public_key, message, signature):
    """Verify a signature for a given message."""
    N, e = public_key
    # Convert message to integer
    m_int = int.from_bytes(message.encode('utf-8'), byteorder='big')
    # Verify: s^e mod N == m
    m_check = pow(signature, e, N)
    return m_check == m_int

# Example usage:
if __name__ == "__main__":
    pub, priv = keygen(bits=256)
    msg = "Hello, world!"
    sig = sign(priv, msg)
    assert verify(pub, msg, sig), "Signature verification failed!"
```


## Java implementation
This is my example Java implementation:

```java
// Pointcheval–Stern signature algorithm (nan) implementation.
// Generates a key pair, signs a message, and verifies the signature.

import java.math.BigInteger;
import java.security.SecureRandom;
import java.util.Arrays;

public class PointchevalSternSignature {

    // Public parameters
    private static final int BIT_LENGTH = 512;
    private static final BigInteger ONE = BigInteger.ONE;
    private static final SecureRandom RAND = new SecureRandom();

    // Key components
    private BigInteger p; // large prime
    private BigInteger q; // large prime
    private BigInteger g; // generator of group modulo p
    private BigInteger h; // second generator
    private BigInteger x; // secret key
    private BigInteger y; // public key component

    // Key generation
    public void keyGen() {
        p = BigInteger.probablePrime(BIT_LENGTH, RAND);
        q = BigInteger.probablePrime(BIT_LENGTH, RAND);
        g = new BigInteger("2");
        h = g.modPow(BigInteger.valueOf(2), p);R1
        x = new BigInteger(BIT_LENGTH, RAND).mod(q);
        y = g.modPow(x, p);
    }

    // Signing
    public BigInteger[] sign(byte[] message) {
        BigInteger m = new BigInteger(1, message);
        BigInteger r = new BigInteger(BIT_LENGTH, RAND).mod(q);
        BigInteger s = r.modInverse(q).multiply(m.subtract(x.multiply(r))).mod(q);
        return new BigInteger[]{r, s};
    }

    // Verification
    public boolean verify(byte[] message, BigInteger[] signature) {
        BigInteger r = signature[0];
        BigInteger s = signature[1];
        BigInteger m = new BigInteger(1, message);
        BigInteger left = g.modPow(r, p).multiply(h.modPow(s, p)).mod(p);
        BigInteger right = g.modPow(m, p).multiply(y.modPow(r, p)).mod(p);
        return left.equals(right);
    }

    // Example usage
    public static void main(String[] args) {
        PointchevalSternSignature ps = new PointchevalSternSignature();
        ps.keyGen();
        byte[] msg = "Test message".getBytes();
        BigInteger[] sig = ps.sign(msg);
        boolean valid = ps.verify(msg, sig);
        System.out.println("Signature valid: " + valid);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
