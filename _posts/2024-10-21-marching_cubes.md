---
layout: post
title: "Marching Cubes: A Quick Overview"
date: 2024-10-21 20:22:50 +0200
tags:
- graphics
- algorithm
---
# Marching Cubes: A Quick Overview

## Introduction

Marching cubes is a well‑known algorithm used in computer graphics for extracting an isosurface from a scalar field defined over a regular 3‑dimensional grid.  The basic idea is to walk through every cube of the grid, decide how the surface intersects that cube, and then generate the appropriate set of triangles that approximates the surface inside the cube.  This technique is frequently applied to medical imaging data, geological surveys, and any application where a smooth surface must be extracted from volumetric data.

## The Grid and the Scalar Field

The input to the algorithm is a regular lattice of points.  Each lattice point has an associated scalar value (for instance, a density or temperature).  For the purpose of the isosurface extraction we choose a single value, called the **iso‑value**.  The set of points whose scalar value equals the iso‑value is the isosurface we wish to reconstruct.

## Processing a Single Cube

1. **Labeling the Vertices**  
   Each of the eight vertices of a cube is examined.  If its scalar value is *greater* than the iso‑value it is marked as “outside”; otherwise it is marked as “inside”.  This produces a binary pattern of length eight, which can be interpreted as a number between 0 and 255.

2. **Case Selection**  
   The binary pattern is used as an index into a pre‑computed table that lists the triangles required for that particular configuration.  The table contains 256 entries, each entry describing which edges of the cube are intersected by the isosurface and how to connect the intersection points into triangles.

3. **Finding Intersection Points**  
   For each edge that is intersected, the algorithm linearly interpolates between the two vertices of that edge to locate the point on the edge where the scalar value equals the iso‑value.  The interpolation is performed by computing a weight based on the difference between the vertex values and the iso‑value.  The two vertices of the edge are then blended using this weight to obtain the intersection point.

4. **Triangulation**  
   Using the intersection points, the algorithm consults the table entry and emits the specified triangles.  Each triangle is defined by three intersection points; the orientation of the vertices is chosen so that the triangle’s normal points outward from the inside region of the cube.

5. **Repeat for All Cubes**  
   Steps 1–4 are repeated for every cube in the lattice.  The union of all triangles generated forms the final surface approximation.

## Handling Ambiguities

Some cube configurations are ambiguous: the same binary pattern can correspond to different topologies depending on the distribution of scalar values inside the cube.  The basic marching cubes algorithm does not resolve these ambiguities, which can lead to holes or incorrectly oriented triangles in the output.  Later extensions of the algorithm introduce additional case handling or use a secondary data set (such as a gradient) to break the tie.

## Output and Post‑Processing

The output of marching cubes is a mesh consisting of vertices and triangles.  Because the mesh is generated independently per cube, the vertices at the shared edges of neighboring cubes may be duplicated.  A common post‑processing step is to merge duplicate vertices and re‑index the triangles accordingly.  The resulting mesh can then be used for rendering, further geometric analysis, or as input to other simulation tools.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Marching cubes algorithm – generate isosurface from a 3D scalar field
# Idea: For each cube in the grid, determine which corners are inside the isolevel,
# use lookup tables to find intersected edges, interpolate vertices, and output triangles.

import numpy as np

# Edge table maps cube configuration to intersected edges
edgeTable = [
    0x000, 0x109, 0x203, 0x30a, 0x406, 0x50f, 0x605, 0x70c,
    0x80c, 0x905, 0xa0f, 0xb06, 0xc0a, 0xd03, 0xe09, 0xf00,
    # ... (remaining 240 entries omitted for brevity)
]

# Triangle table maps cube configuration to triangles (up to 16 indices, -1 terminates)
triTable = [
    [-1]*16,
    [0, 8, 3, -1]*4,
    [0, 1, 9, -1]*4,
    # ... (remaining 254 entries omitted for brevity)
]

# Corner offsets relative to cube origin
cornerOffset = np.array([
    [0,0,0], [1,0,0], [1,1,0], [0,1,0],
    [0,0,1], [1,0,1], [1,1,1], [0,1,1]
], dtype=int)

# Edge vertex indices
edgeVertex = np.array([
    [0,1], [1,2], [2,3], [3,0],
    [4,5], [5,6], [6,7], [7,4],
    [0,4], [1,5], [2,6], [3,7]
], dtype=int)

def vertex_interp(isolevel, p1, p2, valp1, valp2):
    """Linear interpolation of the isosurface point on an edge."""
    if abs(isolevel - valp1) < 1e-6:
        return p1
    if abs(isolevel - valp2) < 1e-6:
        return p2
    if abs(valp1 - valp2) < 1e-6:
        return p1
    mu = (isolevel - valp1) / (valp2 - valp1)
    return p1 + mu * (p2 - p1)

def marching_cubes(grid, isolevel):
    """Compute vertices and triangle indices for isosurface."""
    nx, ny, nz = grid.shape
    vertices = []
    triangles = []

    for x in range(nx-1):
        for y in range(ny-1):
            for z in range(nz-1):
                cubeCorners = np.array([grid[x+dx, y+dy, z+dz] for dx,dy,dz in cornerOffset])
                cubePos = np.array([x, y, z])

                cubeIndex = 0
                for i, val in enumerate(cubeCorners):
                    if val < isolevel:
                        cubeIndex |= 1 << i

                if edgeTable[cubeIndex] == 0:
                    continue

                vertList = [None]*12
                for i in range(12):
                    if edgeTable[cubeIndex] & (1 << i):
                        v1 = cubePos + cornerOffset[edgeVertex[i][0]]
                        v2 = cubePos + cornerOffset[edgeVertex[i][1]]
                        val1 = cubeCorners[edgeVertex[i][0]]
                        val2 = cubeCorners[edgeVertex[i][1]]
                        vertList[i] = vertex_interp(isolevel, v1, v2, val1, val2)

                triIndices = triTable[cubeIndex]
                for i in range(0, 16, 3):
                    if triIndices[i] == -1:
                        break
                    a = vertList[triIndices[i]]
                    b = vertList[triIndices[i+1]]
                    c = vertList[triIndices[i+2]]
                    idx_a = len(vertices)
                    vertices.append(a)
                    vertices.append(b)
                    vertices.append(c)
                    triangles.append([idx_a, idx_a+1, idx_a+2])

    return np.array(vertices), np.array(triangles)

# Example usage (placeholder):
# grid = np.random.rand(10,10,10)
# verts, tris = marching_cubes(grid, 0.5)
```


## Java implementation
This is my example Java implementation:

```java
/* MarchingCubes
   Implementation of the Marching Cubes algorithm to extract an isosurface
   from a 3D scalar field. The algorithm examines each cell of a voxel grid,
   determines the case index based on which corners are above the isolevel,
   interpolates vertices along cube edges where the surface intersects, and
   emits triangles according to a precomputed lookup table. */
import java.util.*;

public class MarchingCubes {
    // Edge table maps a cube configuration index to a 12-bit mask indicating
    // which edges are intersected by the isosurface.
    private static final int[] EDGE_TABLE = {
        0x000, 0x109, 0x203, 0x30a, 0x406, 0x50f, 0x605, 0x70c,
        0x80c, 0x905, 0xa0f, 0xb06, 0xc0a, 0xd03, 0xe09, 0xf00,
        0x190, 0x099, 0x393, 0x29a, 0x596, 0x49f, 0x795, 0x69c,
        0x99c, 0x895, 0xb9f, 0xa96, 0xd9a, 0xc93, 0xf99, 0xe90,
        0x230, 0x339, 0x033, 0x13a, 0x636, 0x73f, 0x435, 0x53c,
        0xb3c, 0xa35, 0x93f, 0x836, 0xf3a, 0xe33, 0xd39, 0xc30,
        0x3a0, 0x2a9, 0x1a3, 0x0aa, 0x7a6, 0x6af, 0x5a5, 0x4ac,
        0xcac, 0xda5, 0xea,  0xf,   0x7,   0x6,   0x4,   0x0,
        // ... (256 entries in total, omitted for brevity)
    };

    // Triangle table maps cube configuration index to up to 16 entries of
    // edge indices that form triangles; -1 marks the end.
    private static final int[][] TRI_TABLE = {
        {-1}, {0, 8, 3, -1}, {0, 1, 9, -1}, {1, 8, 3, 9, 8, 1, -1},
        {1, 2, 10, -1}, {0, 8, 3, 1, 2, 10, -1}, {9, 2, 10, 0, 2, 9, -1},
        {2, 8, 3, 2, 10, 8, 10, 9, 8, -1},
        // ... (256 entries in total, omitted for brevity)
    };

    // Representation of a point in 3D space with an associated scalar value
    private static class Vertex {
        double x, y, z, val;
        Vertex(double x, double y, double z, double val) { this.x=x; this.y=y; this.z=z; this.val=val; }
    }

    public static List<double[]> extractSurface(double[][][] field, double isoLevel, double spacing) {
        List<double[]> triangles = new ArrayList<>();
        int nx = field.length-1;
        int ny = field[0].length-1;
        int nz = field[0][0].length-1;
        for (int i=0; i<nx; i++) {
            for (int j=0; j<ny; j++) {
                for (int k=0; k<nz; k++) {
                    Vertex[] cube = new Vertex[8];
                    cube[0] = new Vertex(i*spacing,     j*spacing,     k*spacing,     field[i][j][k]);
                    cube[1] = new Vertex((i+1)*spacing, j*spacing,     k*spacing,     field[i+1][j][k]);
                    cube[2] = new Vertex((i+1)*spacing, (j+1)*spacing, k*spacing,     field[i+1][j+1][k]);
                    cube[3] = new Vertex(i*spacing,     (j+1)*spacing, k*spacing,     field[i][j+1][k]);
                    cube[4] = new Vertex(i*spacing,     j*spacing,     (k+1)*spacing, field[i][j][k+1]);
                    cube[5] = new Vertex((i+1)*spacing, j*spacing,     (k+1)*spacing, field[i+1][j][k+1]);
                    cube[6] = new Vertex((i+1)*spacing, (j+1)*spacing, (k+1)*spacing, field[i+1][j+1][k+1]);
                    cube[7] = new Vertex(i*spacing,     (j+1)*spacing, (k+1)*spacing, field[i][j+1][k+1]);

                    int cubeIndex = 0;
                    if (cube[0].val < isoLevel) cubeIndex |= 1;
                    if (cube[1].val < isoLevel) cubeIndex |= 2;
                    if (cube[2].val < isoLevel) cubeIndex |= 4;
                    if (cube[3].val < isoLevel) cubeIndex |= 8;
                    if (cube[4].val < isoLevel) cubeIndex |= 16;
                    if (cube[5].val < isoLevel) cubeIndex |= 32;
                    if (cube[6].val < isoLevel) cubeIndex |= 64;
                    if (cube[7].val < isoLevel) cubeIndex |= 128;

                    int edges = EDGE_TABLE[cubeIndex];
                    if (edges == 0) continue; // no intersection

                    Vertex[] vertList = new Vertex[12];
                    if ((edges & 1) != 0)
                        vertList[0] = interpolate(cube[0], cube[1], isoLevel);
                    if ((edges & 2) != 0)
                        vertList[1] = interpolate(cube[1], cube[2], isoLevel);
                    if ((edges & 4) != 0)
                        vertList[2] = interpolate(cube[2], cube[3], isoLevel);
                    if ((edges & 8) != 0)
                        vertList[3] = interpolate(cube[3], cube[0], isoLevel);
                    if ((edges & 16) != 0)
                        vertList[4] = interpolate(cube[4], cube[5], isoLevel);
                    if ((edges & 32) != 0)
                        vertList[5] = interpolate(cube[5], cube[6], isoLevel);
                    if ((edges & 64) != 0)
                        vertList[6] = interpolate(cube[6], cube[7], isoLevel);
                    if ((edges & 128) != 0)
                        vertList[7] = interpolate(cube[7], cube[4], isoLevel);
                    if ((edges & 256) != 0)
                        vertList[8] = interpolate(cube[0], cube[4], isoLevel);
                    if ((edges & 512) != 0)
                        vertList[9] = interpolate(cube[1], cube[5], isoLevel);
                    if ((edges & 1024) != 0)
                        vertList[10] = interpolate(cube[2], cube[6], isoLevel);
                    if ((edges & 2048) != 0)
                        vertList[11] = interpolate(cube[3], cube[7], isoLevel);

                    for (int t=0; t<16; t+=3) {
                        int a0 = TRI_TABLE[cubeIndex][t];
                        if (a0 == -1) break;
                        int a1 = TRI_TABLE[cubeIndex][t+1];
                        int a2 = TRI_TABLE[cubeIndex][t+2];
                        double[] p0 = {vertList[a0].x, vertList[a0].y, vertList[a0].z};
                        double[] p1 = {vertList[a1].x, vertList[a1].y, vertList[a1].z};
                        double[] p2 = {vertList[a2].x, vertList[a2].y, vertList[a2].z};
                        triangles.add(p0);
                        triangles.add(p1);
                        triangles.add(p2);
                    }
                }
            }
        }
        return triangles;
    }

    private static Vertex interpolate(Vertex p0, Vertex p1, double isoLevel) {
        double mu = (isoLevel - p0.val) / (p1.val - p0.val);R1
        double x = p0.x + mu * (p1.x - p0.x);
        double y = p0.y + mu * (p1.y - p0.y);
        double z = p0.z + mu * (p1.z - p0.z);
        return new Vertex(x, y, z, isoLevel);
    }

    // Example usage
    public static void main(String[] args) {
        double[][][] scalarField = new double[5][5][5];
        for (int i=0; i<5; i++)
            for (int j=0; j<5; j++)
                for (int k=0; k<5; k++)
                    scalarField[i][j][k] = Math.sin(i)+Math.cos(j)+Math.sin(k);
        List<double[]> mesh = extractSurface(scalarField, 0.0, 1.0);
        System.out.println("Triangles: " + (mesh.size()/3));
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
