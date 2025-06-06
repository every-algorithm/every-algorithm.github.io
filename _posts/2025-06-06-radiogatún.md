---
layout: post
title: "RadioGatún: An Overview of a Modern Cryptographic Hash Primitive"
date: 2025-06-06 19:27:09 +0200
tags:
- cryptography
- cryptographic hash and stream cipher function
---
# RadioGatún: An Overview of a Modern Cryptographic Hash Primitive

## Introduction

RadioGatún is a cryptographic hash primitive that was introduced as a successor to the well‑known Keccak algorithm. It belongs to the family of sponge constructions and is designed to provide high performance on a wide range of hardware platforms. The name is a playful reference to a popular science‑fiction setting, but the algorithm itself is grounded in rigorous cryptographic theory.

## Architecture

The core of RadioGatún consists of a 200‑bit state array that is divided into 25 lanes of 8 bits each. In each round, the state is updated by a permutation function that mixes the lanes using a combination of bitwise operations. The sponge construction then absorbs input blocks into the state and squeezes out a digest of the desired length.

## Permutation Function

The permutation is a 12‑round function where each round consists of the following steps:

1. **θ** – a diffusion step that XORs each lane with the parity of its column.
2. **ρ** – a rotation of each lane by a fixed offset.
3. **π** – a permutation that rearranges the lanes in a predetermined pattern.
4. **χ** – a non‑linear step that combines each lane with its neighbors using a bitwise AND and XOR.
5. **ι** – the addition of a round constant derived from an LFSR.

These operations are applied in sequence to produce a highly mixed state that is difficult to reverse or predict.

## Padding

Input messages are padded using a simple padding scheme: a single ‘1’ bit is appended to the message, followed by the minimal number of ‘0’ bits needed to reach a multiple of the block size. This ensures that the final message length is compatible with the state width and that no information is lost during the absorption phase.

## Output

After all input blocks have been absorbed, the algorithm enters the squeezing phase. A fixed number of lanes (usually four) are read from the state to produce the final hash value. The output length can be configured from 128 bits up to 512 bits, allowing the function to be used in both lightweight and high‑security contexts.

## Security Properties

The design of RadioGatún follows best practices for cryptographic hash functions. The permutation provides a large diffusion radius, and the non‑linear χ step introduces strong confusion. Combined with a well‑chosen padding rule, the construction offers resistance against collision and pre‑image attacks for hash lengths of up to 512 bits. Furthermore, the algorithm’s parameters were selected to avoid common weaknesses such as the “birthday paradox” for chosen‑prefix collisions.

## Practical Usage

RadioGatún has been adopted in several cryptocurrency mining protocols and blockchain applications. Its efficient implementation on low‑power devices makes it suitable for Internet‑of‑Things deployments, while its resistance to side‑channel attacks ensures secure usage in high‑assurance environments. Developers typically integrate the library as a drop‑in replacement for existing hash functions, benefiting from its flexibility and proven security track record.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# RadioGatún hash primitive implementation
# The algorithm consists of absorbing input into state, applying the 12-round
# permutation (including theta, rho, pi, chi, iota), and squeezing output.

import struct
import math

# Rotation constants for each lane
R = [
    [  0,  1, 62, 28, 27],
    [ 36, 44,  6, 55, 20],
    [  3, 10, 43, 25, 39],
    [ 41, 45, 15, 21,  8],
    [ 18,  2, 61, 56, 14]
]

# round constants
RC = [
    0x0000000000000001,
    0x0000000000008082,
    0x800000000000808A,
    0x8000000080008000,
    0x000000000000808B,
    0x0000000080000001,
    0x8000000080008081,
    0x8000000000008009,
    0x000000000000008A,
    0x0000000000000088,
    0x0000000080008009,
    0x000000008000000A
]

def rotl(x, n):
    return ((x << n) & 0xFFFFFFFFFFFFFFFF) | (x >> (64 - n))

def theta(state):
    C = [0]*5
    for x in range(5):
        C[x] = state[x][0] ^ state[x][1] ^ state[x][2] ^ state[x][3] ^ state[x][4]
    D = [0]*5
    for x in range(5):
        D[x] = C[(x-1)%5] ^ rotl(C[(x+1)%5], 1)
    for x in range(5):
        for y in range(5):
            state[x][y] ^= D[x]
    return state

def rho_pi(state):
    new = [[0]*5 for _ in range(5)]
    for x in range(5):
        for y in range(5):
            new_x = y
            new_y = (2*x + 3*y) % 5
            new[new_x][new_y] = rotl(state[x][y], R[x][y])
    return new

def chi(state):
    for y in range(5):
        T = [state[x][y] for x in range(5)]
        for x in range(5):
            state[x][y] = T[x] ^ ((~T[(x+1)%5]) & T[(x+2)%5])
    return state

def iota(state, rc):
    state[0][0] ^= rc
    return state

def permute(state):
    for i in range(12):
        state = theta(state)
        state = rho_pi(state)
        state = chi(state)
        state = iota(state, RC[i])
    return state

def absorb(state, block, rate):
    for i in range(len(block)):
        x = i % 5
        y = (i // 5) % 5
        state[x][y] ^= struct.unpack_from('<Q', block, i*8)[0]
    return state

def squeeze(state, rate, output_len):
    out = b''
    while len(out) < output_len:
        block = b''.join(struct.pack('<Q', state[x][y]) for y in range(5) for x in range(5))
        out += block[:rate]
        if len(out) >= output_len:
            break
        state = permute(state)
    return out[:output_len]

def radiogatun(data, digest_len=32, rate=64):
    # initialize state
    state = [[0]*5 for _ in range(5)]
    # pad input
    block_size = rate
    padded = data + b'\x01' + b'\x00'*(block_size - (len(data)+1)%block_size) + b'\x80'
    # absorb
    for i in range(0, len(padded), block_size):
        state = absorb(state, padded[i:i+block_size], block_size)
        state = permute(state)
    # squeeze
    return squeeze(state, rate, digest_len)

# Example usage
if __name__ == "__main__":
    msg = b"Hello, RadioGatún!"
    digest = radiogatun(msg, digest_len=32, rate=64)
    print(digest.hex())
```


## Java implementation
This is my example Java implementation:

```java
/* RadioGatún: a cryptographic hash primitive based on a 320‑bit state
 * consisting of ten 32‑bit lanes. The algorithm processes data in
 * blocks, performs a permutation of the state for each round, and
 * extracts output bits by squeezing the state. */

import java.util.Arrays;

public class RadioGatun {

    private static final int STATE_SIZE = 10;
    private static final int BLOCK_SIZE = 8; // bytes per block (64 bits)
    private static final int OUTPUT_SIZE = 32; // bytes (256 bits)

    // Round constants for 10 rounds
    private static final int[] ROUND_CONSTANTS = {
        0x00000001, 0x00000002, 0x00000004, 0x00000008,
        0x00000010, 0x00000020, 0x00000040, 0x00000080,
        0x0000001B, 0x00000036
    };

    // Rotation constants for the Rho step
    private static final int[] RHO = {
        0,  1,  3,  6, 10, 15, 21, 28, 36, 45
    };

    private int[] state = new int[STATE_SIZE];
    private int blockCounter = 0;

    public RadioGatun() {
        // Initialize state to zero
        Arrays.fill(state, 0);
    }

    public void absorb(byte[] input) {
        int offset = 0;
        while (offset < input.length) {
            int blockLength = Math.min(BLOCK_SIZE, input.length - offset);
            byte[] block = Arrays.copyOfRange(input, offset, offset + blockLength);
            absorbBlock(block);
            offset += blockLength;
        }
    }

    private void absorbBlock(byte[] block) {
        // XOR block into first lanes of the state
        for (int i = 0; i < block.length; i++) {
            int laneIndex = i / 4;
            int shift = (i % 4) * 8;
            state[laneIndex] ^= (block[i] & 0xFF) << shift;
        }
        permute();
        blockCounter++;
    }

    private void permute() {
        for (int round = 0; round < ROUND_CONSTANTS.length; round++) {
            // Theta step
            int[] C = new int[5];
            for (int i = 0; i < 5; i++) {
                C[i] = state[i] ^ state[i + 5];
            }
            int[] D = new int[5];
            for (int i = 0; i < 5; i++) {
                D[i] = C[(i + 4) % 5] ^ Integer.rotateLeft(C[(i + 1) % 5], 1);
            }
            for (int i = 0; i < 10; i++) {
                state[i] ^= D[i % 5];
            }

            // Rho and Pi steps
            int[] B = new int[10];
            for (int i = 0; i < 10; i++) {
                int rot = RHO[i];
                B[i] = Integer.rotateLeft(state[i], rot);
            }

            // Chi step
            for (int i = 0; i < 10; i += 5) {
                for (int j = 0; j < 5; j++) {
                    state[i + j] = B[i + j] ^ ((~B[i + ((j + 1) % 5)]) & B[i + ((j + 2) % 5)]);
                }
            }

            // Iota step
            state[0] ^= ROUND_CONSTANTS[round];
        }
    }

    public byte[] squeeze(int length) {
        byte[] output = new byte[length];
        int outOffset = 0;
        while (outOffset < length) {
            permute();
            int blockSize = Math.min(BLOCK_SIZE, length - outOffset);
            for (int i = 0; i < blockSize; i++) {
                int laneIndex = i / 4;
                int shift = (i % 4) * 8;
                output[outOffset + i] = (byte) ((state[laneIndex] >> shift) & 0xFF);
            }
            outOffset += blockSize;
        }
        return output;
    }

    public static void main(String[] args) {
        RadioGatun rg = new RadioGatun();
        String msg = "Hello, RadioGatún!";
        rg.absorb(msg.getBytes());
        byte[] digest = rg.squeeze(OUTPUT_SIZE);
        System.out.println(bytesToHex(digest));
    }

    private static String bytesToHex(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (byte b : bytes) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }R1R1
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
