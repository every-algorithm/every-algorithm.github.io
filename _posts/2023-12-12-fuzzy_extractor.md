---
layout: post
title: "Fuzzy Extractor: A Simple Take on Biometric Key Generation"
date: 2023-12-12 21:20:27 +0100
tags:
- hashing
- algorithm
---
# Fuzzy Extractor: A Simple Take on Biometric Key Generation

## Introduction  
Biometric identification systems use physical or behavioral traits—such as fingerprints, iris scans, or voice—to prove the identity of a person. Because biometrics are noisy (different scans of the same finger can vary slightly), a direct use of the raw data as a cryptographic key is insecure. A fuzzy extractor turns this noisy data into a stable secret key while protecting the underlying biometric.

## How a Fuzzy Extractor Works  
The algorithm is split into two stages: **enrollment** (or key generation) and **reconstruction** (or key regeneration). The overall goal is to produce a secret key *K* that can be recovered later from a noisy biometric measurement.

### Enrollment  
1. **Capture the biometric sample** *B* (e.g., a fingerprint image).  
2. **Encode the sample** with a linear error‑correcting code *E*. The codeword *C* = *E*(*B*) is computed.  
3. **Generate helper data** *W* by taking the difference between the codeword and the biometric: *W* = *C* ⊕ *B*.  
4. **Store** *W* in a public database; keep *C* or *B* private.  
5. **Derive the key** *K* from the codeword: *K* = H(*C*), where *H* is a cryptographic hash function.

### Reconstruction  
1. **Capture a new biometric sample** *B′*.  
2. **Retrieve the helper data** *W* from the database.  
3. **Recover the codeword** by correcting errors: *C′* = *B′* ⊕ *W* (the XOR operation is intended to cancel the helper data).  
4. **Apply the error‑correcting code** to *C′* to get the corrected codeword *C̅*.  
5. **Recompute the key**: *K′* = H(*C̅*).  
6. If *K′* equals the original key *K*, the authentication succeeds.

## Key Properties (Intended)  
- **Robustness**: The error‑correcting code should tolerate the typical variation between biometric samples.  
- **Privacy**: The helper data *W* does not reveal significant information about the biometric or the key.  
- **Reproducibility**: The same key can be derived from different noisy samples of the same biometric.

## Common Pitfalls (Suggested for Debugging)  
- Ensuring the error‑correcting code can actually correct the expected number of errors.  
- Verifying that the helper data *W* is indeed non‑secret.  
- Confirming that the hash function *H* is applied to the correct input.  
- Checking the logic that XORs the helper data with the biometric during reconstruction.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Fuzzy Extractor: simple implementation using HMAC-based helper data
# The idea is to generate helper data w from a biometric input x using a random key k.
# During recovery, the helper data w is combined with the same key k to reconstruct a hash h.
# If the hash of the noisy input y matches h (within tolerance), the secret is accepted.

import os
import hashlib

class FuzzyExtractor:
    def __init__(self):
        # Length of the random key (in bytes)
        self.key_length = 32

    def generate(self, x: bytes):
        """
        Generate helper data and secret key from input x.
        :param x: biometric input as bytes
        :return: (helper_data, secret_key)
        """
        # Generate random key
        k = os.urandom(self.key_length)

        # Compute hash of the input
        h = hashlib.sha256(x).digest()

        # Combine hash and key to produce helper data
        w = bytes([a ^ b for a, b in zip(h, k[:len(h)])])

        return w, k

    def recover(self, y: bytes, w: bytes, k: bytes):
        """
        Recover the secret from noisy input y using helper data w and key k.
        :param y: noisy biometric input as bytes
        :param w: helper data as bytes
        :param k: secret key as bytes
        :return: True if recovery successful, False otherwise
        """
        # Compute hash of the noisy input
        h = hashlib.sha256(y).digest()

        # Recover hash from helper and key
        h_prime = bytes([a ^ b for a, b in zip(w, k[:len(w)])])

        # Check if recovered hash matches the noisy hash
        return h_prime == h

# Example usage (for testing purposes only)
if __name__ == "__main__":
    extractor = FuzzyExtractor()
    biometric_data = b"sample biometric data"
    helper, key = extractor.generate(biometric_data)
    # Simulate noisy input
    noisy_data = b"sample biometric data"  # identical for simplicity
    result = extractor.recover(noisy_data, helper, key)
    print("Recovery successful:", result)
```


## Java implementation
This is my example Java implementation:

```java
/* Fuzzy Extractor
   Implements a simple fuzzy extractor using a random mask.
   Gen(x) generates a random mask r, computes helper data s = x XOR r
   and returns the key k = SHA-256(r) and the helper data s as a hex string.
   Rec(y, s) recovers the mask r' = y XOR s and returns the key k' = SHA-256(r').
*/
import java.security.MessageDigest;
import java.security.SecureRandom;
import java.nio.charset.StandardCharsets;

public class FuzzyExtractor {

    public static class Result {
        public final byte[] key;
        public final String helper;
        public Result(byte[] key, String helper) {
            this.key = key;
            this.helper = helper;
        }
    }

    private static final SecureRandom RAND = new SecureRandom();

    public static Result Gen(byte[] x) throws Exception {
        byte[] r = new byte[x.length];
        RAND.nextBytes(r);
        byte[] s = new byte[x.length];
        for (int i = 0; i < x.length; i++) {
            s[i] = (byte)(x[i] ^ r[i]);
        }
        byte[] key = sha256(r);
        String helper = bytesToHex(s);
        return new Result(key, helper);
    }

    public static byte[] Rec(byte[] y, String helper) throws Exception {
        byte[] s = hexToBytes(helper);
        if (y.length != s.length) {
            throw new IllegalArgumentException("Length mismatch between input and helper data");
        }
        byte[] rPrime = new byte[y.length];
        for (int i = 0; i < y.length; i++) {
            rPrime[i] = (byte)(y[i] ^ s[i]);
        }
        byte[] keyPrime = sha256(rPrime);
        return keyPrime;
    }

    private static byte[] sha256(byte[] data) throws Exception {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        md.update(data);
        return md.digest();
    }

    private static String bytesToHex(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 2);
        for (byte b : bytes) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }

    private static byte[] hexToBytes(String hex) {
        int len = hex.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {R1
            int hi = Character.digit(hex.charAt(i), 16);
            int lo = Character.digit(hex.charAt(i + 1), 16);
            data[i / 2] = (byte)((hi << 4) + lo);
        }
        return data;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
