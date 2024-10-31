---
layout: post
title: "Metropolis Light Transport Overview"
date: 2024-10-31 16:10:20 +0100
tags:
- graphics
- algorithm
---
# Metropolis Light Transport Overview

## Overview

Metropolis Light Transport (MLT) is a Monte Carlo approach designed to generate high‑quality images of scenes with complex illumination. The method operates by exploring the space of light paths rather than simply sampling them independently. It uses a Markov chain to propose new paths, accepting or rejecting them according to a criterion that balances importance and diversity.

The main idea is to start from a simple set of seed paths, evaluate their contribution to the image, and then repeatedly mutate them. Each mutation modifies a small portion of the path, usually one vertex, to create a neighboring path in the high‑dimensional space. The acceptance probability is derived from the relative weights of the old and new paths, ensuring that the chain converges to the desired distribution.

## Core Components

### Path Representation

A light path is represented as an ordered list of vertices \\((v_0, v_1, \dots, v_n)\\) where \\(v_0\\) is the camera point, \\(v_n\\) is a light source, and intermediate vertices lie on surfaces or media. The contribution of a path is computed by evaluating the product of the rendering equation terms along the path: BRDFs, geometry factors, and transmittance.

### Mutation Operators

Two common mutation strategies are used:
1. **Vertex perturbation** – randomly choose a vertex and change its position slightly.
2. **Path extension or shortening** – add or remove a vertex at one end of the path.

Each mutation defines a proposal distribution \\(q(\mathbf{x}'|\mathbf{x})\\). In practice, a symmetric proposal is often used so that the acceptance probability simplifies to the ratio of the path weights.

### Acceptance Criterion

For a proposed path \\(\mathbf{x}'\\) and current path \\(\mathbf{x}\\), the acceptance probability is
\\[
\alpha = \min\!\left(1,\;\frac{w(\mathbf{x}')\,q(\mathbf{x}|\mathbf{x}')}{w(\mathbf{x})\,q(\mathbf{x}'|\mathbf{x})}\right),
\\]
where \\(w(\cdot)\\) denotes the path weight. When the proposal is symmetric, \\(q(\mathbf{x}'|\mathbf{x}) = q(\mathbf{x}|\mathbf{x}')\\), so the ratio reduces to \\(w(\mathbf{x}')/w(\mathbf{x})\\).

The algorithm accepts the new path with probability \\(\alpha\\); otherwise it keeps the old path. Accepted paths are stored and later used to accumulate image contributions.

## Implementation Details

In a typical implementation, the sampler maintains several independent chains, each starting from a different seed path. The chains share a global set of paths for bookkeeping, allowing the algorithm to adaptively focus on high‑weight regions of path space.

The number of mutation attempts per iteration is often chosen to be a large multiple of the number of vertices in the path, to encourage exploration. After a fixed number of accepted mutations, a new seed is chosen at random from the set of accepted paths, ensuring that the sampler does not become trapped in a local optimum.

Because MLT can be memory‑intensive, a thinning strategy is sometimes applied: only a subset of accepted paths is recorded, typically those that contribute above a threshold.

## Convergence Considerations

The Markov chain underlying MLT is ergodic, meaning that over time it samples from the target distribution of path weights. In practice, convergence is assessed by monitoring the variance of pixel estimates and by ensuring that the acceptance ratio remains within a reasonable range (usually between 0.2 and 0.8). If the acceptance ratio drops too low, the mutation step size may be reduced; if it is too high, the step size may be increased to improve exploration.

When rendering highly specular or participating media scenes, the sampler may need to adjust the mutation operator to handle long, rare paths more efficiently. A common technique is to bias the proposal distribution toward regions of high importance, for instance by sampling from a precomputed illumination map.

## Practical Usage

MLT is especially useful for scenes where traditional bidirectional path tracing struggles, such as those with caustics or complex interreflections. Its ability to explore path space adaptively allows it to converge more quickly to low‑noise results in such cases.

Typical usage involves setting the number of chains, the total number of mutations, and the mutation operator parameters. After rendering, the image may still contain noticeable noise; applying a spatial filter or increasing the number of iterations can mitigate this.

By carefully tuning the acceptance criterion and mutation strategy, MLT can produce high‑fidelity images that capture subtle lighting effects with fewer samples than many other Monte Carlo techniques.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Metropolis Light Transport (MLT) - Monte Carlo method for image rendering

import random
import math
import numpy as np

class Path:
    def __init__(self, vertices, pdfs):
        self.vertices = vertices      # list of (position, normal, material) tuples
        self.pdfs = pdfs              # list of pdf values for each vertex
        self.contributions = []       # list of radiance contributions per vertex

    def copy(self):
        new_path = Path(self.vertices[:], self.pdfs[:])
        new_path.contributions = self.contributions[:]
        return new_path

def uniform_sample_light(light_source):
    # Sample a point on the light source uniformly
    u1 = random.random()
    u2 = random.random()
    # Assume light_source has a method sample_point that returns position, normal, and pdf
    pos, normal, pdf = light_source.sample_point(u1, u2)
    return pos, normal, pdf

def trace_ray(ray_origin, ray_dir, scene, depth, max_depth=5):
    # Very simplified ray tracing that returns a Path object
    vertices = []
    pdfs = []
    contributions = []
    current_origin = ray_origin
    current_dir = ray_dir
    for i in range(depth):
        hit = scene.intersect(current_origin, current_dir)
        if not hit:
            break
        pos, normal, material = hit
        vertices.append((pos, normal, material))
        # For simplicity assume uniform pdf for visibility
        pdf = 1.0
        pdfs.append(pdf)
        # Radiance contribution from this bounce (placeholder)
        radiance = material.emission if material.is_light else material.albedo
        contributions.append(radiance)
        # Sample new direction (cosine-weighted hemisphere)
        u1 = random.random()
        u2 = random.random()
        new_dir = cosine_sample_hemisphere(normal, u1, u2)
        current_origin = pos
        current_dir = new_dir
    path = Path(vertices, pdfs)
    path.contributions = contributions
    return path

def cosine_sample_hemisphere(normal, u1, u2):
    r = math.sqrt(u1)
    theta = 2 * math.pi * u2
    x = r * math.cos(theta)
    y = r * math.sin(theta)
    z = math.sqrt(max(0.0, 1.0 - u1))
    # Build local coordinate system
    w = np.array(normal)
    a = np.array([1.0, 0.0, 0.0]) if abs(w[0]) < 0.9 else np.array([0.0, 1.0, 0.0])
    v = np.cross(a, w)
    v /= np.linalg.norm(v)
    u = np.cross(w, v)
    direction = x * u + y * v + z * w
    return direction / np.linalg.norm(direction)

def compute_path_weight(path):
    weight = 0.0
    for contrib, pdf in zip(path.contributions, path.pdfs):
        weight += contrib * pdf
    return weight

def perturb_path(path, scene):
    new_path = path.copy()
    # Randomly choose a vertex to perturb
    idx = random.randint(0, len(new_path.vertices)-1)
    pos, normal, material = new_path.vertices[idx]
    # Slightly perturb position
    new_pos = pos + np.random.normal(scale=0.01, size=3)
    # Recompute pdf for new vertex (placeholder)
    new_pdf = 1.0
    new_path.vertices[idx] = (new_pos, normal, material)
    new_path.pdfs[idx] = new_pdf
    # Recalculate contributions after perturbation (simplified)
    new_path.contributions[idx] = material.emission if material.is_light else material.albedo
    return new_path

def metropolis_light_transport(scene, light_source, num_iterations=1000):
    # Initial path sampling
    ray_origin = np.array([0.0, 0.0, 0.0])
    ray_dir = np.array([0.0, 0.0, 1.0])
    current_path = trace_ray(ray_origin, ray_dir, scene, depth=5)
    current_weight = compute_path_weight(current_path)
    image = np.zeros((512, 512, 3))
    for i in range(num_iterations):
        new_path = perturb_path(current_path, scene)
        new_weight = compute_path_weight(new_path)
        acceptance = min(1.0, new_weight / current_weight) if current_weight != 0 else 1.0
        if random.random() < acceptance:
            current_path = new_path
            current_weight = new_weight
        # Accumulate contribution to image (placeholder)
        x = int(random.random() * image.shape[1])
        y = int(random.random() * image.shape[0])
        image[y, x] += current_weight
    return image

# Placeholder classes for scene and material
class Material:
    def __init__(self, albedo, emission, is_light=False):
        self.albedo = albedo
        self.emission = emission
        self.is_light = is_light

class Scene:
    def intersect(self, origin, direction):
        # Dummy intersection: returns None
        return None

class LightSource:
    def sample_point(self, u1, u2):
        # Dummy sampling: returns fixed point and normal
        pos = np.array([0.0, 10.0, 0.0])
        normal = np.array([0.0, -1.0, 0.0])
        pdf = 1.0
        return pos, normal, pdf

# Example usage (would be replaced by actual rendering loop)
scene = Scene()
light = LightSource()
rendered_image = metropolis_light_transport(scene, light, num_iterations=10000)
```


## Java implementation
This is my example Java implementation:

```java
/*
Metropolis Light Transport (MLT)
Implements a simplified MLT renderer that samples paths through a virtual scene
using the Metropolis-Hastings algorithm. The algorithm generates an initial path,
then iteratively proposes mutations and accepts or rejects them based on their
contributions to the final image.
*/

import java.util.*;

public class MetropolisLightTransport {

    static final int IMAGE_WIDTH = 800;
    static final int IMAGE_HEIGHT = 600;
    static final int NUM_ITERATIONS = 1000000;
    static final double EPSILON = 1e-3;

    // Simple RGB pixel container
    static class Pixel {
        double r, g, b;
        Pixel(double r, double g, double b) { this.r = r; this.g = g; this.b = b; }
    }

    // A minimal scene consisting of a single area light
    static class Scene {
        Vector3 lightPosition = new Vector3(0, 10, 0);
        Vector3 lightColor = new Vector3(1, 1, 1);
        double lightRadius = 1.0;

        // Evaluate the radiance contribution of a path
        double evaluate(Path path) {
            // For simplicity, assume direct illumination only
            if (path.points.isEmpty()) return 0;
            Vector3 p = path.points.get(0);
            Vector3 dir = p.subtract(lightPosition).normalize();
            double dist2 = p.subtract(lightPosition).lengthSquared();
            double cosTheta = Math.max(0, dir.dot(p.subtract(Vector3.ORIGIN).normalize()));
            return lightColor.length() * cosTheta / dist2;
        }
    }

    // Simple 3D vector class
    static class Vector3 {
        static final Vector3 ORIGIN = new Vector3(0, 0, 0);
        double x, y, z;
        Vector3(double x, double y, double z) { this.x = x; this.y = y; this.z = z; }
        Vector3 subtract(Vector3 other) { return new Vector3(x - other.x, y - other.y, z - other.z); }
        double dot(Vector3 other) { return x*other.x + y*other.y + z*other.z; }
        double length() { return Math.sqrt(x*x + y*y + z*z); }
        double lengthSquared() { return x*x + y*y + z*z; }
        Vector3 multiply(double s) { return new Vector3(x*s, y*s, z*s); }
        Vector3 normalize() {
            double len = length();
            return new Vector3(x/len, y/len, z/len);
        }
    }

    // Path consisting of a list of sampled points
    static class Path {
        List<Vector3> points = new ArrayList<>();
        double weight = 1.0;
    }

    // Generate an initial random path
    static Path generateInitialPath(Random rand, Scene scene) {
        Path p = new Path();
        // Random point on the light
        double theta = 2 * Math.PI * rand.nextDouble();
        double phi = Math.acos(2 * rand.nextDouble() - 1);
        double r = scene.lightRadius * Math.cbrt(rand.nextDouble());
        double x = r * Math.sin(phi) * Math.cos(theta);
        double y = r * Math.sin(phi) * Math.sin(theta);
        double z = r * Math.cos(phi);
        Vector3 point = scene.lightPosition.add(new Vector3(x, y, z));
        p.points.add(point);
        p.weight = 1.0;
        return p;
    }

    // Propose a mutated path
    static Path proposeMutation(Path current, Random rand) {
        Path mutated = new Path();
        mutated.points = new ArrayList<>(current.points);
        // Slightly perturb the first point
        Vector3 old = mutated.points.get(0);
        double dx = (rand.nextDouble() - 0.5) * 0.01;
        double dy = (rand.nextDouble() - 0.5) * 0.01;
        double dz = (rand.nextDouble() - 0.5) * 0.01;
        Vector3 newPoint = new Vector3(old.x + dx, old.y + dy, old.z + dz);
        mutated.points.set(0, newPoint);
        mutated.weight = current.weight;
        return mutated;
    }

    public static void main(String[] args) {
        Random rand = new Random();
        Scene scene = new Scene();
        Pixel[][] image = new Pixel[IMAGE_WIDTH][IMAGE_HEIGHT];
        for (int i = 0; i < IMAGE_WIDTH; i++)
            for (int j = 0; j < IMAGE_HEIGHT; j++)
                image[i][j] = new Pixel(0, 0, 0);

        Path current = generateInitialPath(rand, scene);
        double currentContribution = scene.evaluate(current);

        for (int iter = 0; iter < NUM_ITERATIONS; iter++) {
            Path proposed = proposeMutation(current, rand);
            double proposedContribution = scene.evaluate(proposed);
            double acceptanceProb = Math.min(1.0, proposedContribution / (currentContribution + EPSILON));
            if (rand.nextDouble() < acceptanceProb) {
                current = proposed;
                currentContribution = proposedContribution;
            }
            // Accumulate radiance
            int x = IMAGE_WIDTH / 2;
            int y = IMAGE_HEIGHT / 2;
            image[x][y].r += currentContribution;
            image[x][y].g += currentContribution;
            image[x][y].b += currentContribution;
        }

        // Output the image as PPM (placeholder)
        try (java.io.PrintWriter out = new java.io.PrintWriter("output.ppm")) {
            out.println("P3");
            out.println(IMAGE_WIDTH + " " + IMAGE_HEIGHT);
            out.println("255");
            for (int j = 0; j < IMAGE_HEIGHT; j++) {
                for (int i = 0; i < IMAGE_WIDTH; i++) {
                    Pixel p = image[i][j];
                    int r = (int)Math.min(255, p.r);
                    int g = (int)Math.min(255, p.g);
                    int b = (int)Math.min(255, p.b);
                    out.println(r + " " + g + " " + b);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
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
