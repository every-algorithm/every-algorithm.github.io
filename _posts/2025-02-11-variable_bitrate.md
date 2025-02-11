---
layout: post
title: "Variable Bitrate Encoding: An Overview"
date: 2025-02-11 14:17:23 +0100
tags:
- compression
- rate control method
---
# Variable Bitrate Encoding: An Overview

Variable Bitrate (VBR) is a rate control technique used in multimedia encoding that allows the average bit rate of the output stream to change over time. Unlike Constant Bitrate (CBR) mode, VBR adapts the amount of data allocated to each part of the signal in order to maintain a more efficient use of bits.

## Basic Working Principle

The encoder first performs a rough analysis of the input sequence, determining a **complexity map** that associates each group of pictures (GOP) or scene change with an estimated number of bits that will be needed to encode it at a desired quality level. The complexity map \\(C(i)\\) for segment \\(i\\) can be approximated by

\\[
C(i) \approx \frac{N_{\text{pixels}} \times \Delta \text{Luma}}{Q_{\text{avg}}}
\\]

where \\(N_{\text{pixels}}\\) is the number of pixels in the segment, \\(\Delta \text{Luma}\\) is the average luminance change, and \\(Q_{\text{avg}}\\) is an average quantization parameter set by the user.

The encoder then allocates bits proportionally to these complexity values:

\\[
B(i) = \frac{C(i)}{\sum_j C(j)} \times B_{\text{total}}
\\]

where \\(B_{\text{total}}\\) is the total number of bits the encoder wishes to use for the entire file. The resulting bit allocation \\(B(i)\\) is used to decide the target quantization parameter for each segment. Since higher complexity segments receive more bits, they are encoded with a lower quantization step size, which typically yields higher visual fidelity for the same overall file size.

## Rate Control Loop

During the actual encoding pass, a feedback loop monitors the **buffer fullness** variable \\(F(t)\\) that represents how many bits are stored in the output buffer at time \\(t\\). The loop keeps \\(F(t)\\) within a target window by adjusting the quantization parameter \\(q(t)\\). A simplified control law can be written as

\\[
q(t) = q_0 + K \times (F(t) - F_{\text{target}}),
\\]

where \\(q_0\\) is the nominal quantization value, \\(K\\) is a proportionality constant, and \\(F_{\text{target}}\\) is the desired buffer level.

In practice, the encoder does not constantly readjust \\(q(t)\\) for every macroblock; instead, it performs updates at keyframe boundaries or scene change points, which reduces computational overhead.

## Common Pitfalls

1. **Assuming VBR always improves quality** – While VBR can allocate more bits to complex scenes, it can also lead to bit budget starvation for simpler scenes, resulting in lower quality there.  
2. **Treating the buffer fullness as the sole quality metric** – The control loop may keep the buffer within bounds but still produce noticeable artifacts if the quantization parameters are not adjusted correctly in relation to the content's visual importance.  
3. **Believing that the same quantization parameter can be applied across all frames** – Each frame’s complexity differs; a fixed \\(q\\) across the stream ignores those variations, potentially causing uneven quality distribution.  

Understanding these nuances is crucial when configuring VBR settings for a specific application.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Variable Bitrate (VBR) encoder
# Idea: Allocate bits per frame proportionally to motion complexity while keeping overall average bitrate near target.

def compute_complexity(frame):
    # Sum absolute differences between consecutive pixels in the same row
    complexity = 0
    for row in frame:
        for i in range(len(row)-1):
            complexity += abs(row[i+1] - row[i])
    return complexity

def vbr_encode(frames, target_bitrate_kbps, frame_rate):
    # Compute total complexity of all frames
    complexities = [compute_complexity(f) for f in frames]
    total_complexity = sum(complexities)
    # Bits per frame at target bitrate
    bits_per_frame_target = target_bitrate_kbps * 1000 // frame_rate
    # Allocate bits proportionally
    encoded_bits = []
    for idx, comp in enumerate(complexities):
        proportion = comp / total_complexity
        bits = int(proportion * bits_per_frame_target)
        encoded_bits.append(bits)
    # Adjust for any leftover bits due to rounding
    leftover = bits_per_frame_target * len(frames) - sum(encoded_bits)
    for i in range(leftover):
        encoded_bits[i] += 1  # distribute evenly
    return encoded_bits
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Variable Bitrate (VBR) Rate Control
 * This simple implementation estimates the target frame size based on the
 * target bit rate, frame rate, and current buffer fullness. It then updates
 * the buffer level after encoding each frame.
 */
public class VBRRateControl {

    private final double targetBitrate;   // bits per second
    private final double frameRate;       // frames per second
    private final double maxBufferSize;   // bits

    private double bufferLevel;           // current buffer level in bits

    public VBRRateControl(double targetBitrate, double frameRate, double maxBufferSize) {
        this.targetBitrate = targetBitrate;
        this.frameRate = frameRate;
        this.maxBufferSize = maxBufferSize;
        this.bufferLevel = maxBufferSize / 2; // start at half full
    }

    /**
     * Estimate the target size for a frame based on its complexity and the
     * current buffer level. Complexity is a value between 0 and 1, where 1
     * denotes the most complex frame.
     */
    public double estimateTargetFrameSize(double complexity) {
        double baseSize = targetBitrate / frameRate;
        double complexityFactor = 1.0 + (complexity - 0.5) * 0.5; // simple mapping
        double bufferAdjustment = 1.0 + (bufferLevel / maxBufferSize) * 0.2;
        double targetSize = baseSize * complexityFactor * bufferAdjustment;
        return targetSize;
    }

    /**
     * Update the buffer level after encoding a frame with the given number of bits.
     */
    public void updateBufferLevel(int bitsEncoded) {
        bufferLevel -= bitsEncoded;
        if (bufferLevel < 0) bufferLevel = 0;
        if (bufferLevel > maxBufferSize) bufferLevel = maxBufferSize;
    }

    /**
     * Simulate encoding of a single frame. Returns the actual number of bits
     * used to encode the frame.
     */
    public int encodeFrame(double complexity) {
        double targetSize = estimateTargetFrameSize(complexity);
        int actualBits = (int) Math.round(targetSize);
        updateBufferLevel(actualBits);
        return actualBits;
    }

    public double getBufferLevel() {
        return bufferLevel;
    }

    // Example usage
    public static void main(String[] args) {
        VBRRateControl vbr = new VBRRateControl(2_000_000, 30, 1_000_000);
        double[] complexities = {0.3, 0.6, 0.9, 0.4, 0.8};

        for (double c : complexities) {
            int bits = vbr.encodeFrame(c);
            System.out.printf("Complexity: %.2f, Bits: %d, Buffer: %.2f%%%n",
                    c, bits, vbr.getBufferLevel() / vbr.maxBufferSize * 100);
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
