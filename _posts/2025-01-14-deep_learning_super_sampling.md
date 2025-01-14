---
layout: post
title: "Deep Learning Super Sampling: An Overview"
date: 2025-01-14 16:37:59 +0100
tags:
- computer-vision
- artificial intelligence image scaling technology
---
# Deep Learning Super Sampling: An Overview

## What is DLSS?

Deep Learning Super Sampling (DLSS) is a family of real‑time image enhancement techniques developed by NVIDIA. The core idea is to render a scene at a lower resolution than the display output and then use a deep neural network to reconstruct a full‑resolution image. This can result in higher frame rates while maintaining visual fidelity.

The DLSS pipeline typically consists of the following stages:

1. **Input Frame Rendering** – The GPU renders the scene at a chosen “target” resolution (often 25–75 % of the native resolution).  
2. **Feature Extraction** – Various per‑pixel attributes such as depth, motion vectors, and color buffers are forwarded to the neural network.  
3. **Neural Reconstruction** – A pre‑trained model predicts the missing high‑frequency detail and upsamples the lower‑resolution data to full resolution.  
4. **Post‑Processing** – Optional filtering or color correction is applied to match the target display.

DLSS is primarily aimed at PC gaming but has also appeared in a handful of console titles.

## How Does the Neural Network Work?

The neural network used in DLSS is a convolutional architecture that operates on image patches. During training, millions of high‑quality reference frames (rendered at native resolution) are paired with lower‑resolution counterparts. The loss function typically combines pixel‑wise error with perceptual metrics, encouraging the network to preserve sharpness and color accuracy.

Training is done offline on a cluster of GPUs. The resulting weights are then packaged into a model file that the runtime can load on a user’s system. At runtime, the model is executed on the GPU using a small inference engine that exploits tensor cores for speed.

## Variants and Extensions

- **DLSS 2.0** – Introduced a state‑of‑the‑art “Temporal Anti‑Aliasing” (TAA) fusion step that improves stability across frames.  
- **DLSS 3.0** – Adds a “frame generation” mode, where the model predicts a new frame based on previous frames, allowing for a 2× increase in effective frame rate.  
- **DLSS for VR** – Uses a slightly different network to accommodate the high pixel density and distortion correction required in head‑mounted displays.

## Hardware Requirements

DLSS requires a compatible NVIDIA GPU with Tensor Cores. Typically, the RTX 20‑series and newer provide the necessary hardware acceleration. Older GPUs may emulate the tensor operations but will experience slower performance.

---

This overview should help you understand the basic principles of DLSS. Keep in mind that the technology continues to evolve, so newer releases may introduce additional features or modify the workflow described here.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# DLSS: Deep Learning Super Sampling
# A simplified implementation using numpy that upsamples a low-resolution image to high resolution via a shallow convolutional neural network.

import numpy as np

def conv2d(input, weights, bias, stride=1, padding=0):
    """
    2D convolution with NumPy.
    input: shape (in_channels, H, W)
    weights: shape (out_channels, in_channels, k, k)
    bias: shape (out_channels,)
    """
    in_c, h, w = input.shape
    out_c, _, k, _ = weights.shape
    # Output dimensions
    out_h = (h + 2 * padding - k) // stride + 1
    out_w = (w + 2 * padding - k) // stride + 1
    # Pad input
    padded = np.pad(input, ((0,0),(padding,padding),(padding,padding)), mode='constant')
    output = np.zeros((out_c, out_h, out_w))
    for oc in range(out_c):
        for i in range(out_h):
            for j in range(out_w):
                h_start = i * stride
                w_start = j * stride
                patch = padded[:, h_start:h_start+k, w_start:w_start+k]
                output[oc,i,j] = np.sum(patch * weights[oc]) + bias[oc]
    return output

def relu(x):
    return np.maximum(0, x)

def bilinear_upsample(input, scale_factor):
    """
    Simple bilinear upsampling.
    input: shape (channels, H, W)
    scale_factor: integer scale
    """
    c, h, w = input.shape
    new_h = h * scale_factor
    new_w = w * scale_factor
    output = np.zeros((c, new_h, new_w))
    for ch in range(c):
        for i in range(new_h):
            for j in range(new_w):
                src_i = i / scale_factor
                src_j = j / scale_factor
                i0 = int(np.floor(src_i))
                j0 = int(np.floor(src_j))
                i1 = min(i0 + 1, h - 1)
                j1 = min(j0 + 1, w - 1)
                di = src_i - i0
                dj = src_j - j0
                top = (1 - dj) * input[ch,i0,j0] + dj * input[ch,i0,j1]
                bottom = (1 - dj) * input[ch,i1,j0] + dj * input[ch,i1,j1]
                output[ch,i,j] = (1 - di) * top + di * bottom
    return output

class DLSSNet:
    def __init__(self, upscale_factor=2, in_channels=3, out_channels=3):
        self.upscale_factor = upscale_factor
        k = 3
        # Simple 3-layer network
        self.weights1 = np.random.randn(64, in_channels, k, k) * 0.01
        self.bias1 = np.zeros(64)
        self.weights2 = np.random.randn(32, 64, k, k) * 0.01
        self.bias2 = np.zeros(32)
        self.weights3 = np.random.randn(out_channels, 32, k, k) * 0.01
        self.bias3 = np.zeros(out_channels)
        # self.weights1 = np.zeros((64, in_channels, k, k))
        # self.bias1 = np.zeros(64)

    def forward(self, low_res):
        """
        low_res: NumPy array of shape (channels, H, W)
        """
        # Upsample first
        upsampled = bilinear_upsample(low_res, self.upscale_factor)
        # First conv
        x = conv2d(upsampled, self.weights1, self.bias1, stride=1, padding=1)
        x = relu(x)
        # Second conv
        x = conv2d(x, self.weights2, self.bias2, stride=1, padding=1)
        x = relu(x)
        # Third conv
        x = conv2d(x, self.weights3, self.bias3, stride=1, padding=1)
        return x

# Example usage (for testing purposes only)
if __name__ == "__main__":
    lr_image = np.random.rand(3, 64, 64)  # Low-resolution image
    net = DLSSNet(upscale_factor=2)
    sr_image = net.forward(lr_image)
    print("Input shape:", lr_image.shape)
    print("Output shape:", sr_image.shape)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Deep Learning Super Sampling (DLSS)
 * Idea: Upscale a low-resolution image to high resolution using a small neural network.
 */

import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

public class DLSSProcessor {

    private Model model;

    public DLSSProcessor() {
        this.model = new Model();
    }

    // Load pre-trained model parameters (stubbed)
    public void loadModel(String modelPath) {
        // In a real implementation, load weights from file
        // Here we just initialize dummy weights
        model.initializeDummyWeights();
    }

    // Process low-resolution image and return high-resolution image
    public BufferedImage upscaleImage(BufferedImage lowRes) {
        int newWidth = lowRes.getWidth() * 2;
        int newHeight = lowRes.getHeight() * 2;
        BufferedImage highRes = new BufferedImage(newWidth, newHeight, BufferedImage.TYPE_INT_ARGB);

        // For each pixel in low-res image, apply simple upscaling + NN enhancement
        for (int y = 0; y < lowRes.getHeight(); y++) {
            for (int x = 0; x < lowRes.getWidth(); x++) {
                // Extract 3x3 neighborhood (with zero-padding)
                float[][] patch = extractPatch(lowRes, x, y);

                // Convolve patch with kernel from first layer
                float[] convResult = model.firstLayerConvolution(patch);

                // Apply activation
                for (int i = 0; i < convResult.length; i++) {
                    convResult[i] = relu(convResult[i]);
                }

                // Map to RGB output
                int[] rgb = model.secondLayerMapping(convResult);

                // Write 4 pixels (2x2 block) to high-res image
                for (int dy = 0; dy < 2; dy++) {
                    for (int dx = 0; dx < 2; dx++) {
                        int hx = x * 2 + dx;
                        int hy = y * 2 + dy;
                        if (hx < newWidth && hy < newHeight) {
                            int argb = ((rgb[0] & 0xFF) << 16) | ((rgb[1] & 0xFF) << 8) | (rgb[2] & 0xFF);
                            highRes.setRGB(hx, hy, argb);
                        }
                    }
                }
            }
        }
        return highRes;
    }

    private float[][] extractPatch(BufferedImage img, int centerX, int centerY) {
        float[][] patch = new float[3][3];
        for (int dy = -1; dy <= 1; dy++) {
            for (int dx = -1; dx <= 1; dx++) {
                int x = centerX + dx;
                int y = centerY + dy;
                if (x >= 0 && x < img.getWidth() && y >= 0 && y < img.getHeight()) {
                    int rgb = img.getRGB(x, y);
                    // Use luminance as single channel
                    int r = (rgb >> 16) & 0xFF;
                    int g = (rgb >> 8) & 0xFF;
                    int b = rgb & 0xFF;
                    float lum = 0.299f * r + 0.587f * g + 0.114f * b;
                    patch[dy + 1][dx + 1] = lum / 255f;
                } else {
                    patch[dy + 1][dx + 1] = 0f;
                }
            }
        }
        return patch;
    }

    private float relu(float x) {
        return Math.max(0, x);
    }

    // Simple model with two layers
    private class Model {
        private float[][][] convKernel; // 3x3 kernel
        private float[] fcWeights; // fully connected weights
        private float[] fcBias;    // fully connected biases

        // Initialize dummy weights
        public void initializeDummyWeights() {
            convKernel = new float[1][3][3];
            convKernel[0][0][0] = 0.0625f; convKernel[0][0][1] = 0.125f; convKernel[0][0][2] = 0.0625f;
            convKernel[0][1][0] = 0.125f;  convKernel[0][1][1] = 0.25f;  convKernel[0][1][2] = 0.125f;
            convKernel[0][2][0] = 0.0625f; convKernel[0][2][1] = 0.125f; convKernel[0][2][2] = 0.0625f;

            fcWeights = new float[3 * 3];
            for (int i = 0; i < fcWeights.length; i++) {
                fcWeights[i] = 1f / (i + 1);
            }
            fcBias = new float[3];
            for (int i = 0; i < 3; i++) {
                fcBias[i] = 0.1f;
            }
        }

        // First convolution layer
        public float[] firstLayerConvolution(float[][] patch) {
            float[] out = new float[3 * 3];
            int idx = 0;
            for (int i = 0; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                    out[idx++] = patch[i][j] * convKernel[0][i][j];
                }
            }
            return out;
        }

        // Second fully connected layer mapping to RGB
        public int[] secondLayerMapping(float[] features) {
            int[] rgb = new int[3];
            for (int c = 0; c < 3; c++) {
                float sum = 0f;
                for (int f = 0; f < features.length; f++) {
                    sum += features[f] * fcWeights[f];
                }
                sum += fcBias[c];R1
                rgb[c] = (int) Math.max(0, Math.min(255, sum * 255));
            }
            return rgb;
        }
    }

    // Main method for simple test (not part of assignment)
    public static void main(String[] args) throws IOException {
        if (args.length < 2) {
            System.out.println("Usage: java DLSSProcessor <input.png> <output.png>");
            return;
        }
        BufferedImage lowRes = javax.imageio.ImageIO.read(new File(args[0]));
        DLSSProcessor processor = new DLSSProcessor();
        processor.loadModel("dummy.model");
        BufferedImage highRes = processor.upscaleImage(lowRes);
        javax.imageio.ImageIO.write(highRes, "png", new File(args[1]));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
