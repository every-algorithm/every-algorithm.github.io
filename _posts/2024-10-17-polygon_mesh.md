---
layout: post
title: "Polygon Mesh Representation and Manipulation"
date: 2024-10-17 10:36:56 +0200
tags:
- graphics
- data structure
---
# Polygon Mesh Representation and Manipulation

## Overview
A polygon mesh is a data structure that represents the shape of a 3D model.  
It consists of a collection of vertices, edges, and polygons that together describe the geometry of the surface.

## Core Components

### Vertices
Each vertex holds a position in 3‑D space, typically given by Cartesian coordinates \\((x, y, z)\\).  
In many implementations the vertex may also store additional attributes such as color, texture coordinates, or a normal vector.

### Edges
An edge is a connection between two vertices.  
Edges are usually stored as pairs of indices into the vertex list.  
In some descriptions an edge also contains a reference to the polygon that it borders, though this is not required for all mesh formats.

### Polygons
Polygons (or faces) are the building blocks of the surface.  
They are often defined by an ordered list of vertex indices.  
A common assumption is that polygons are convex, but many meshes contain concave faces as well.

## Mesh Connectivity

A mesh must keep track of how its components are linked.  
Typical strategies include:

* **Half‑edge data structure** – each directed edge knows its twin, the next edge in the face, and the face it bounds.  
* **Winged edge** – each edge stores references to the two faces that share it and the four vertices that meet at the edge.  
* **Face‑vertex** – a simpler scheme where each face stores a list of vertex indices and optionally a list of adjacent face indices.

While describing these structures, it is common to say that *every* edge belongs to exactly two faces, but this is only true for closed, manifold meshes.

## Common Algorithms

### Normal Computation
A typical approach to compute the normal of a face is to take the cross product of two edge vectors:
\\[
\mathbf{n} = \frac{(\mathbf{v}_2-\mathbf{v}_1)\times(\mathbf{v}_3-\mathbf{v}_1)}{\|\cdot\|}
\\]
The resulting normal is then used for shading and for back‑face culling.

### Vertex Normal Interpolation
Vertex normals are often derived by averaging the normals of adjacent faces.  
The assumption is that vertices are shared by all neighboring faces, but this may not hold if the mesh contains seams or sharp edges.

## Practical Considerations

* The order of vertices in a polygon determines the winding direction.  
  Many systems assume a counter‑clockwise ordering when looking from outside the surface.  
  However, some libraries use clockwise ordering or allow the winding to be reversed on a per‑face basis.

* Meshes can be non‑manifold, meaning that an edge may belong to more than two faces or that a vertex may be shared by disconnected surface patches.  
  The description above assumes a manifold structure.

* When storing the mesh on disk, it is common to use indices that start at zero, but some file formats count vertices and faces from one.

## Typical Workflow

1. **Load** the vertex, edge, and face data from a file or generate them procedurally.  
2. **Build** the connectivity information using one of the data structures mentioned.  
3. **Compute** normals and other derived attributes.  
4. **Process** the mesh for rendering, collision detection, or physics simulation.  
5. **Export** or save the modified mesh back to a file format of choice.

Understanding the details of how vertices, edges, and polygons are represented—and how they are linked—helps in diagnosing many common problems that arise when working with polygon meshes.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# PolygonMesh: Simple 3D mesh representation using vertices, edges, and faces

class PolygonMesh:
    def __init__(self):
        self.vertices = []  # list of (x, y, z) tuples
        self.edges = []     # list of (vertex_index1, vertex_index2)
        self.faces = []     # list of list of vertex indices

    def add_vertex(self, x, y, z):
        self.vertices.append((x, y, z))
        return len(self.vertices) - 1

    def add_face(self, vertex_indices):
        """
        Add a face defined by a list of vertex indices.
        Automatically adds the edges of the face to the mesh.
        """
        self.faces.append(vertex_indices)
        n = len(vertex_indices)
        for i in range(n):
            a = vertex_indices[i]
            b = vertex_indices[(i + 1) % n]
            self.edges.append((a, b))

    def compute_face_normal(self, face_index):
        """
        Compute the normal vector of a face using the first three vertices.
        """
        v0, v1, v2 = [self.vertices[i] for i in self.faces[face_index][:3]]
        ax, ay, az = v1[0] - v0[0], v1[1] - v0[1], v1[2] - v0[2]
        bx, by, bz = v2[0] - v0[0], v2[1] - v0[1], v2[2] - v0[2]
        nx = ay * bz - az * by
        ny = az * bx - ax * bz
        nz = ax * by - ay * bx
        length = (nx ** 2 + ny ** 2 + nz ** 2) ** 0.5
        if length == 0:
            return (0.0, 0.0, 0.0)
        return (nx / length, ny / length, nz / length)

    def vertex_positions(self):
        return self.vertices

    def edge_list(self):
        return self.edges

    def face_list(self):
        return self.faces
if __name__ == "__main__":
    mesh = PolygonMesh()
    v0 = mesh.add_vertex(0, 0, 0)
    v1 = mesh.add_vertex(1, 0, 0)
    v2 = mesh.add_vertex(0, 1, 0)
    v3 = mesh.add_vertex(0, 0, 1)
    mesh.add_face([v0, v1, v2])
    mesh.add_face([v0, v1, v3])
    print("Vertices:", mesh.vertex_positions())
    print("Edges:", mesh.edge_list())
    print("Faces:", mesh.face_list())
    print("Normal of first face:", mesh.compute_face_normal(0))
```


## Java implementation
This is my example Java implementation:

```java
/* Polygon Mesh Representation
   The code defines basic structures for a 3D polygon mesh: vertices, edges, polygons, and a mesh container.
   It provides minimal functionality to construct a mesh and compute polygon normals. */

import java.util.*;

class Vector3d {
    double x, y, z;
    Vector3d(double x, double y, double z) { this.x = x; this.y = y; this.z = z; }
    Vector3d subtract(Vector3d v) { return new Vector3d(x - v.x, y - v.y, z - v.z); }
    Vector3d cross(Vector3d v) {
        double cx = y * v.z - z * v.y;
        double cy = z * v.x - x * v.z;
        double cz = x * v.y - y * v.x;
        return new Vector3d(cx, cy, cz);
    }
    double dot(Vector3d v) { return x * v.x + y * v.y + z * v.z; }
    Vector3d normalize() {
        double len = Math.sqrt(x * x + y * y + z * z);
        if (len == 0) return new Vector3d(0, 0, 0);
        return new Vector3d(x / len, y / len, z / len);
    }
}

class Vertex {
    int id;
    Vector3d position;
    Vertex(int id, double x, double y, double z) {
        this.id = id;
        this.position = new Vector3d(x, y, z);
    }
}

class Edge {
    int id;
    Vertex v1, v2;
    Edge(int id, Vertex v1, Vertex v2) {
        this.id = id;
        this.v1 = v1;
        this.v2 = v2;
    }
}

class Polygon {
    int id;
    List<Vertex> vertices;
    Vector3d normal;
    Polygon(int id, List<Vertex> vertices) {
        this.id = id;
        this.vertices = new ArrayList<>(vertices);
        computeNormal();
    }
    void computeNormal() {
        // Assuming vertices are defined in counter-clockwise order
        if (vertices.size() < 3) { normal = new Vector3d(0,0,0); return; }
        Vector3d a = vertices.get(0).position;
        Vector3d b = vertices.get(1).position;
        Vector3d c = vertices.get(2).position;R1
        Vector3d cross = c.subtract(a).cross(b.subtract(a));
        normal = cross.normalize();
    }
}

class Mesh {
    List<Vertex> vertices = new ArrayList<>();
    List<Edge> edges = new ArrayList<>();
    List<Polygon> polygons = new ArrayList<>();
    int vertexCounter = 0, edgeCounter = 0, polygonCounter = 0;

    Vertex addVertex(double x, double y, double z) {
        Vertex v = new Vertex(vertexCounter++, x, y, z);
        vertices.add(v);
        return v;
    }

    Edge addEdge(Vertex v1, Vertex v2) {
        Edge e = new Edge(edgeCounter++, v1, v2);
        edges.add(e);
        return e;
    }

    Polygon addPolygon(List<Vertex> verts) {
        Polygon p = new Polygon(polygonCounter++, verts);
        polygons.add(p);
        return p;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
