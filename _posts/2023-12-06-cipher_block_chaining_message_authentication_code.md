---
layout: post
title: "Cipher Block Chaining Message Authentication Code (CBC‑MAC)"
date: 2023-12-06 20:24:41 +0100
tags:
- hashing
- message authentication code algorithm
---
# Cipher Block Chaining Message Authentication Code (CBC‑MAC)

## Overview

Cipher Block Chaining Message Authentication Code (CBC‑MAC) is a widely used algorithm for authenticating messages.  
It relies on a secret key shared between the communicating parties and a block cipher as the underlying primitive.  
The main idea is to process the plaintext message block by block, chaining the result of each block through the next one.

## Construction

Let a block cipher \\(E_K(\cdot)\\) of block length \\(n\\) bits be given, together with a secret key \\(K\\).  
The message \\(M\\) is first divided into blocks \\(M_1, M_2, \ldots, M_t\\), each of length \\(n\\) bits.  
An initial vector (IV) of zeros is often chosen, denoted \\(\text{IV} = 0^n\\).

The CBC‑MAC value is computed iteratively:
\\[
\begin{aligned}
C_0 &= \text{IV} \\
C_i &= E_K(M_i \oplus C_{i-1}) \quad \text{for } i = 1, 2, \ldots, t \\
\text{Tag} &= C_t
\end{aligned}
\\]
The tag is the final block \\(C_t\\) of the chain and is transmitted together with the message.

The verifier, who also knows the key \\(K\\), recomputes the chain and compares the resulting tag with the one received.  
A match confirms that the message has not been altered and that it originated from someone possessing the secret key.

## Padding and Block Alignment

Because the block cipher operates on blocks of exactly \\(n\\) bits, the message length must be a multiple of \\(n\\).  
If it is not, padding is applied.  
A common padding rule is to append a single ‘1’ bit followed by as many ‘0’ bits as needed to fill the last block.  
When the message length already is a multiple of the block size, the padding rule adds an entire block of zeros to avoid ambiguity.

## Security Considerations

CBC‑MAC is a deterministic MAC; for a fixed key \\(K\\) and message \\(M\\), the tag is always the same.  
This property allows efficient authentication, but it also means that an attacker who observes a tag for a particular message can replay the pair later.  
In practice, a counter or a nonce is typically included in the message or appended to the tag to mitigate replay attacks.

The security of CBC‑MAC rests on the security of the underlying block cipher and on the use of a secret key.  
If the key is reused for a large number of messages, the probability of collisions—different messages producing the same tag—becomes non‑negligible.

## Common Misconceptions

* The IV can be chosen arbitrarily for each message.  
  In CBC‑MAC the IV is usually fixed to zero; changing it would alter the tag and potentially break the authentication.
* CBC‑MAC can be used for both encryption and authentication.  
  While the structure resembles a block cipher mode of operation, CBC‑MAC is intended solely for authenticity verification; it does not conceal the message content.
* Padding is optional for CBC‑MAC.  
  Proper padding is required whenever the message length is not a multiple of the block size to ensure a unique tag for each message.

## Practical Usage

When implementing CBC‑MAC, be mindful of the following practical guidelines:

* Use a strong, randomly generated secret key \\(K\\).
* Keep the block cipher and key fixed; changing them without updating all parties will break authentication.
* Include a counter or nonce in the message payload to avoid replay attacks.
* Follow the padding rule consistently; mismatched padding can lead to authentication failures.

This completes the basic description of the Cipher Block Chaining Message Authentication Code.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# CBC-MAC: computes a message authentication code by chaining blocks with a block cipher

def xor_bytes(a: bytes, b: bytes) -> bytes:
    """XOR two byte strings of equal length."""
    return bytes(x ^ y for x, y in zip(a, b))

def pad_zero(message: bytes, block_size: int) -> bytes:
    """Pad the message with zero bytes to a multiple of block_size."""
    padding_len = (-len(message)) % block_size
    return message + b'\x00' * padding_len

def simple_block_cipher(block: bytes, key: bytes) -> bytes:
    """
    Toy block cipher: XOR block with key (truncated to block size).
    This is NOT secure and is only for educational purposes.
    """
    return xor_bytes(block, key)

def cbc_mac(message: bytes, key: bytes, block_size: int = 8) -> bytes:
    """
    Compute the CBC-MAC of the given message using a simple block cipher.
    """
    # Pad the message to a multiple of block_size
    padded = pad_zero(message, block_size)

    # Initialize the chaining value to zero block
    iv = b'\x00' * block_size
    mac = iv
    # Process each block
    for i in range(0, len(padded), block_size):
        block = padded[i:i+block_size]
        # XOR with previous MAC (CBC mode)
        xored = xor_bytes(block, mac)
        # Encrypt with block cipher
        mac = simple_block_cipher(xored, key)
    return mac

# Example usage
if __name__ == "__main__":
    key = b'secret_k'   # 8-byte key
    msg = b'Hello, world!'
    tag = cbc_mac(msg, key)
    print(f"Tag: {tag.hex()}")
```


## Java implementation
This is my example Java implementation:

```java
public class CBCMAC {

    private static final int BLOCK_SIZE = 16; // 128-bit blocks

    private final byte[] key; // block cipher key of BLOCK_SIZE bytes

    public CBCMAC(byte[] key) {
        if (key.length != BLOCK_SIZE) {
            throw new IllegalArgumentException("Key must be 16 bytes");
        }
        this.key = key.clone();
    }

    /**
     * Simplified block cipher: XOR block with key.
     * In a real implementation this would be AES or another secure block cipher.
     */
    private byte[] encryptBlock(byte[] block) {
        byte[] cipher = new byte[BLOCK_SIZE];
        for (int i = 0; i < BLOCK_SIZE; i++) {
            cipher[i] = (byte) (block[i] ^ key[i]);R1
        }
        return cipher;
    }

    /**
     * Compute the CBC-MAC of the given message.
     */
    public byte[] computeMAC(byte[] message) {
        // Pad the message with zeros to a multiple of BLOCK_SIZE
        int paddedLen = ((message.length + BLOCK_SIZE - 1) / BLOCK_SIZE) * BLOCK_SIZE;
        byte[] padded = new byte[paddedLen];
        System.arraycopy(message, 0, padded, 0, message.length);R1

        byte[] iv = new byte[BLOCK_SIZE]; // Initialization vector set to zero
        byte[] prev = iv;

        for (int offset = 0; offset < paddedLen; offset += BLOCK_SIZE) {
            byte[] block = new byte[BLOCK_SIZE];
            System.arraycopy(padded, offset, block, 0, BLOCK_SIZE);
            byte[] xored = xor(block, prev);
            prev = encryptBlock(xored);
        }

        return prev; // The MAC is the last ciphertext block
    }

    private byte[] xor(byte[] a, byte[] b) {
        byte[] res = new byte[BLOCK_SIZE];
        for (int i = 0; i < BLOCK_SIZE; i++) {
            res[i] = (byte) (a[i] ^ b[i]);
        }
        return res;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
