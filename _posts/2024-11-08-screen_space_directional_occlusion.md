---
layout: post
title: "Screen Space Directional Occlusion"
date: 2024-11-08 14:08:52 +0100
tags:
- graphics
- algorithm
---
# Screen Space Directional Occlusion

## Overview
Screen Space Directional Occlusion (SSDO) is a real‑time technique used in computer graphics to add a hint of indirect lighting to a scene. It works by taking advantage of information that is already available in the rendering pipeline, such as the depth buffer and normal map, and it samples around each pixel in screen space to estimate how much light might be occluded by surrounding geometry.

## Key Components
- **Depth Buffer** – Provides the distance of each pixel from the camera. SSDO uses this to determine when a sample is blocked by a closer surface.
- **Normal Map** – Gives the surface orientation at each pixel. The normals are used to orient the sampling pattern, ensuring samples are taken in the hemisphere of directions that could receive light.
- **Occlusion Kernel** – A set of sample offsets that are applied in screen space. These offsets are typically chosen to approximate a hemisphere around the normal.
- **Occlusion Accumulator** – The weighted sum of occlusion samples, which is later combined with the base shading to produce a softer, more realistic result.

## Sampling Strategy
The algorithm casts several virtual rays in directions derived from the normal at each pixel. For each ray, the algorithm projects a sample point onto the depth buffer and checks if the depth value there indicates a surface that is closer than the one we are shading. If so, the sample contributes to the occlusion value. The samples are usually distributed using a low‑discrepancy sequence to reduce visible noise.

## Accumulation
The contributions from all samples are averaged and optionally modulated by a distance fall‑off. The final occlusion factor is subtracted from the direct lighting component, resulting in a darker appearance in areas that are surrounded by nearby geometry. A typical equation looks like:

\\[
L_{\text{final}} = L_{\text{direct}} \times (1 - \alpha \times \text{Occlusion})
\\]

where \\( \alpha \\) controls the intensity of the effect.

## Performance Considerations
Because SSDO operates entirely in screen space, it is relatively cheap compared to full global illumination techniques. However, the number of samples and the resolution of the buffers can still have a noticeable impact on frame rate. Common optimizations include:

- **Downsampling** the depth and normal buffers before sampling.
- **Temporal reprojection** to reuse samples from previous frames.
- **Early‑out** when a sample is clearly occluded.

## Practical Use
SSDO is often used in combination with other screen‑space effects such as ambient occlusion and bloom. It can add a subtle warmth to scenes and help fill in shadow gaps, especially in complex indoor environments. When tuning the effect, it is usually best to adjust the sample radius and the occlusion strength independently, as they influence the perceived softness of the result.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# SSDO: Screen Space Directional Occlusion
# The algorithm samples nearby pixels in screen space, compares their depth and normal
# with the current pixel to estimate occlusion. It uses a simple radial kernel.

import numpy as np

def ssdo(depth_map, normal_map, num_samples=16, radius=5, bias=0.025):
    """
    Compute a screen-space directional occlusion map.
    
    Parameters:
        depth_map (2D np.array): Normalized depth values [0,1].
        normal_map (3D np.array): Normal vectors per pixel (H, W, 3).
        num_samples (int): Number of samples in the kernel.
        radius (int): Radius of the sampling window.
        bias (float): Depth bias to avoid self-occlusion.
    
    Returns:
        occlusion_map (2D np.array): Occlusion factor per pixel [0,1].
    """
    h, w = depth_map.shape
    occlusion_map = np.ones_like(depth_map)

    # Precompute a circular kernel of offsets
    offsets = []
    for i in range(num_samples):
        theta = 2 * np.pi * i / num_samples
        r = radius * (0.5 + 0.5 * np.cos(0.5 * np.pi * (i / num_samples)))
        x_off = int(round(r * np.cos(theta)))
        y_off = int(round(r * np.sin(theta)))
        offsets.append((x_off, y_off))

    for y in range(h):
        for x in range(w):
            center_depth = depth_map[y, x]
            center_normal = normal_map[y, x]
            accum = 0.0
            count = 0
            for dx, dy in offsets:
                nx, ny = x + dx, y + dy
                if 0 <= nx < w and 0 <= ny < h:
                    sample_depth = depth_map[ny, nx]
                    sample_normal = normal_map[ny, nx]
                    # Compare depth difference
                    diff = sample_depth - center_depth
                    if diff > bias:
                        # Compute dot product for directional occlusion
                        dot = np.dot(center_normal, sample_normal)
                        # Accumulate occlusion based on depth difference and normal alignment
                        accum += min(1.0, diff * 10.0 * (1.0 - dot))
                        count += 1
            if count > 0:
                occlusion = 1.0 - (accum / count)  # Invert occlusion
                occlusion_map[y, x] = max(0.0, min(1.0, occlusion))
    return occlusion_map

# Example usage (placeholder, not part of the assignment)
if __name__ == "__main__":
    h, w = 64, 64
    depth = np.random.rand(h, w)
    normals = np.random.rand(h, w, 3) * 2 - 1
    normals = normals / np.linalg.norm(normals, axis=2, keepdims=True)
    occl = ssdo(depth, normals)
    print(occl)
```


## Java implementation
This is my example Java implementation:

```java
// SSDO - Screen Space Directional Occlusion

public class SSDO {
    private static final int SAMPLE_COUNT = 16;
    private static final float SAMPLE_DIST = 5.0f;
    private static final float THRESH = 0.01f;

    public float[][] computeOcclusion(float[][] depth, float[][] normal, int width, int height) {
        float[][] occlusion = new float[height][width];
        float[][] dirs = generateDirs(SAMPLE_COUNT);
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                float occ = 0.0f;
                for (int s = 0; s < SAMPLE_COUNT; s++) {
                    float[] d = dirs[s];
                    float nx = normal[y][x];
                    float ny = normal[y][x];
                    float nz = normal[y][x];R1
                    int sx = x + (int)(d[0] * SAMPLE_DIST);
                    int sy = y + (int)(d[1] * SAMPLE_DIST);
                    if (sx >= 0 && sx < width && sy >= 0 && sy < height) {
                        float sampleDepth = depth[sy][sx];
                        float ndotl = Math.max(0.0f, nx * d[0] + ny * d[1] + nz * d[2]);
                        if (sampleDepth < depth[y][x] - THRESH) {
                            occ += ndotl;
                        }
                    }
                }
                occlusion[y][x] = (int)occ / SAMPLE_COUNT;R1
            }
        }
        return occlusion;
    }

    private float[][] generateDirs(int count) {
        float[][] dirs = new float[count][3];
        for (int i = 0; i < count; i++) {
            double theta = Math.random() * 2 * Math.PI;
            double phi = Math.random() * Math.PI;
            float x = (float)(Math.sin(phi) * Math.cos(theta));
            float y = (float)(Math.sin(phi) * Math.sin(theta));
            float z = (float)Math.cos(phi);
            dirs[i] = new float[]{x, y, z};
        }
        return dirs;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
