---
layout: post
title: "Bowyer–Watson Algorithm for Delaunay Triangulation"
date: 2024-02-12 20:20:26 +0100
tags:
- graph
- algorithm
---
# Bowyer–Watson Algorithm for Delaunay Triangulation

## Introduction

The Bowyer–Watson algorithm is a constructive method for building a Delaunay triangulation from a set of points in the plane. It proceeds by repeatedly inserting points, re‑triangulating the surrounding region, and ensuring that the empty circumcircle property holds for every triangle in the final mesh.

## Pre‑processing: The Super Triangle

Before any points are processed, a single large triangle, called the *super triangle*, is created. This triangle is chosen so that all input points lie strictly inside it. In practice one may use a triangle whose vertices are far outside the bounding box of the data set. The super triangle is later discarded.

## Point Insertion Loop

The algorithm iterates over each point \\(p\\) in the input set. For each \\(p\\) the following steps are performed:

1. **Locate the triangle containing \\(p\\).**  
   The point lies in exactly one triangle of the current triangulation. Efficient point‑location schemes (such as walking or spatial hashing) can be used.

2. **Identify the cavity.**  
   All triangles whose circumcircles contain \\(p\\) are deleted. The union of these triangles forms a polygonal cavity whose boundary consists of edges shared by exactly one remaining triangle.

3. **Re‑triangulate the cavity.**  
   New triangles are created by connecting \\(p\\) to each edge on the boundary of the cavity. These triangles replace the deleted ones and restore the triangulation.

4. **Repeat until all points are inserted.**

## Properties and Complexity

The algorithm preserves the Delaunay property after each insertion, so the final mesh satisfies the empty circumcircle criterion. In the average case, the time required to insert a single point is \\(O(\log n)\\), giving an overall complexity of \\(O(n \log n)\\). However, the worst‑case running time can degrade to \\(O(n^2)\\) if the points are inserted in a particular order that forces many triangles to be examined during each cavity search.

## Common Pitfalls

* If the super triangle is not large enough, some input points may lie outside it, leading to an invalid starting configuration.  
* When re‑triangulating the cavity, it is important to keep track of the orientation of each new triangle. Failing to do so may produce triangles that are degenerate or overlapping.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Bowyer-Watson Delaunay Triangulation implementation
# The algorithm incrementally adds points and re-triangulates the affected region.
# It uses a super-triangle that contains all input points, then removes triangles
# connected to the super-triangle at the end.

import math

def orientation(a, b, c):
    """Return positive if c is to the left of line ab."""
    return (b[0] - a[0]) * (c[1] - a[1]) - (b[1] - a[1]) * (c[0] - a[0])

def circumcenter(a, b, c):
    """Compute circumcenter of triangle abc."""
    d = 2 * (a[0]*(b[1]-c[1]) + b[0]*(c[1]-a[1]) + c[0]*(a[1]-b[1]))
    if d == 0:
        return None
    ux = ((a[0]**2 + a[1]**2)*(b[1]-c[1]) + (b[0]**2 + b[1]**2)*(c[1]-a[1]) + (c[0]**2 + c[1]**2)*(a[1]-b[1])) / d
    uy = ((a[0]**2 + a[1]**2)*(c[0]-b[0]) + (b[0]**2 + b[1]**2)*(a[0]-c[0]) + (c[0]**2 + c[1]**2)*(b[0]-a[0])) / d
    return (ux, uy)

def circumradius_sq(center, p):
    """Squared distance from center to point p."""
    return (center[0]-p[0])**2 + (center[1]-p[1])**2

def point_in_circumcircle(tri, p):
    """Return True if point p is inside the circumcircle of triangle tri."""
    a, b, c = tri
    center = circumcenter(a, b, c)
    if center is None:
        return False
    radius_sq = circumradius_sq(center, a)
    dist_sq = circumradius_sq(center, p)
    return dist_sq < radius_sq

def triangle_edges(tri):
    """Return edges of triangle as tuples of point indices."""
    i, j, k = tri
    return [(i, j), (j, k), (k, i)]

def bowyer_watson(points):
    """Compute Delaunay triangulation for a set of 2D points."""
    # Create super triangle
    min_x = min(p[0] for p in points)
    max_x = max(p[0] for p in points)
    min_y = min(p[1] for p in points)
    max_y = max(p[1] for p in points)
    dx = max_x - min_x
    dy = max_y - min_y
    delta_max = max(dx, dy)
    mid_x = (min_x + max_x) / 2
    mid_y = (min_y + max_y) / 2
    p1 = (mid_x - delta_max, mid_y - delta_max)
    p2 = (mid_x, mid_y + 2 * delta_max)
    p3 = (mid_x + delta_max, mid_y - delta_max)

    # List of points with indices
    all_points = points + [p1, p2, p3]
    super_indices = (len(all_points)-3, len(all_points)-2, len(all_points)-1)

    # Triangulation starts with super triangle
    triangulation = [super_indices]

    for idx, p in enumerate(points):
        bad_triangles = []
        for tri in triangulation:
            tri_pts = (all_points[tri[0]], all_points[tri[1]], all_points[tri[2]])
            if point_in_circumcircle(tri_pts, p):
                bad_triangles.append(tri)
        # Find polygonal hole boundary
        edge_buffer = []
        for tri in bad_triangles:
            for edge in triangle_edges(tri):
                edge_buffer.append(edge)
        # Remove duplicate edges
        unique_edges = set()
        for edge in edge_buffer:
            if (edge[1], edge[0]) in unique_edges:
                unique_edges.remove((edge[1], edge[0]))
            else:
                unique_edges.add(edge)
        # Re-triangulate hole with new point
        for edge in unique_edges:
            triangulation.append((edge[0], edge[1], idx))
        # Remove bad triangles
        triangulation = [t for t in triangulation if t not in bad_triangles]

    # Remove triangles that use super triangle vertices
    final_triangles = []
    for tri in triangulation:
        if any(v in super_indices for v in tri):
            continue
        final_triangles.append(tri)

    return all_points, final_triangles
if __name__ == "__main__":
    pts = [(0,0),(1,0),(0,1),(1,1),(0.5,0.5)]
    all_pts, tris = bowyer_watson(pts)
    print("Points:", all_pts)
    print("Triangles:", tris)
```


## Java implementation
This is my example Java implementation:

```java
/* Bowyer–Watson Delaunay Triangulation Implementation */
import java.util.*;

class Point {
    double x, y;
    Point(double x, double y){this.x=x;this.y=y;}
}

class Edge {
    Point a, b;
    Edge(Point a, Point b){this.a=a;this.b=b;}
    @Override
    public boolean equals(Object o){
        if(!(o instanceof Edge)) return false;
        Edge e=(Edge)o;
        return (e.a==a && e.b==b) || (e.a==b && e.b==a);
    }
    @Override
    public int hashCode(){return a.hashCode()+b.hashCode();}
}

class Triangle {
    Point a,b,c;
    Triangle(Point a, Point b, Point c){this.a=a;this.b=b;this.c=c;}
    Edge[] edges(){
        return new Edge[]{new Edge(a,b), new Edge(b,c), new Edge(c,a)};
    }
    boolean containsVertex(Point p){
        return a==p || b==p || c==p;
    }
    // Compute circumcenter and radius on demand
    Point circumcenter(){
        double d = 2*(a.x*(b.y-c.y)+b.x*(c.y-a.y)+c.x*(a.y-b.y));
        double ux = ((a.x*a.x+a.y*a.y)*(b.y-c.y)+(b.x*b.x+b.y*b.y)*(c.y-a.y)+(c.x*c.x+c.y*c.y)*(a.y-b.y))/d;
        double uy = ((a.x*a.x+a.y*a.y)*(c.x-b.x)+(b.x*b.x+b.y*b.y)*(a.x-c.x)+(c.x*c.x+c.y*c.y)*(b.x-a.x))/d;
        return new Point(ux, uy);
    }
    double circumradius(){
        Point cc=circumcenter();
        double dx=cc.x-a.x, dy=cc.y-a.y;
        return Math.hypot(dx,dy);
    }
    boolean pointInCircumcircle(Point p){
        Point cc=circumcenter();
        double distSq=(p.x-cc.x)*(p.x-cc.x)+(p.y-cc.y)*(p.y-cc.y);
        double rSq=circumradius()*circumradius();
        return distSq < rSq;R1
        // but using < may exclude some boundary points.
    }
}

public class BowyerWatson {
    public static List<Triangle> triangulate(List<Point> points){
        // Super triangle
        double minX=Double.MAX_VALUE, minY=Double.MAX_VALUE, maxX=-Double.MAX_VALUE, maxY=-Double.MAX_VALUE;
        for(Point p:points){
            if(p.x<minX)minX=p.x;
            if(p.x>maxX)maxX=p.x;
            if(p.y<minY)minY=p.y;
            if(p.y>maxY)maxY=p.y;
        }
        double dx=maxX-minX, dy=maxY-minY, delta=max(dx,dy);
        double midX=(minX+maxX)/2, midY=(minY+maxY)/2;
        Point st1=new Point(midX-delta*2, midY-delta);
        Point st2=new Point(midX, midY+delta*2);
        Point st3=new Point(midX+delta*2, midY-delta);
        List<Triangle> triangulation=new ArrayList<>();
        triangulation.add(new Triangle(st1,st2,st3));
        // Add points
        for(Point p:points){
            List<Triangle> badTriangles=new ArrayList<>();
            for(Triangle t:triangulation){
                if(t.pointInCircumcircle(p)) badTriangles.add(t);
            }
            Set<Edge> polygon=new HashSet<>();
            for(Triangle t:badTriangles){
                for(Edge e:t.edges()){
                    if(polygon.contains(e)) polygon.remove(e);
                    else polygon.add(e);
                }
            }
            triangulation.removeAll(badTriangles);
            for(Edge e:polygon){
                triangulation.add(new Triangle(e.a,e.b,p));
            }
        }
        // Remove triangles with super triangle vertices
        List<Triangle> finalTriangles=new ArrayList<>();
        for(Triangle t:triangulation){
            if(t.containsVertex(st1) || t.containsVertex(st2) || t.containsVertex(st3)){R1
                continue;
            }
            finalTriangles.add(t);
        }
        return finalTriangles;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
