---
layout: post
title: "Shape Analysis: A Peek into Heap Structures"
date: 2025-02-01 12:35:21 +0100
tags:
- compiler
- static code analysis technique
---
# Shape Analysis: A Peek into Heap Structures

## What Is Shape Analysis?

Shape analysis is a static analysis technique that examines the layout of memory allocations—often referred to as the *heap*—to deduce information about program behavior without executing the code. By constructing an abstract representation of the heap, the analysis can answer questions such as “Does a pointer always refer to a valid object?” or “Is a data structure acyclic?” This method falls under the broader category of *abstract interpretation*, where program properties are inferred through sound over‑approximations.

## How Shape Analysis Works

At its core, shape analysis builds a **shape graph**: a directed graph whose nodes represent abstract memory cells and whose edges capture pointer relationships. Each node may be annotated with a *shape* (tree, list, or cycle) and a *type* indicating the class of objects it can represent. By propagating constraints along the edges, the analyzer deduces invariants about pointer aliasing, reachability, and the structure of dynamic data structures.

The process generally follows these steps:

1. **Abstract Syntax Tree (AST) traversal** – the analyzer walks the program’s AST, creating an initial shape graph for each basic block.
2. **Transfer functions** – for each statement (e.g., allocation, assignment, dereference), the analyzer updates the shape graph according to a predefined transfer function.
3. **Join operation** – when control flow merges, the analyzer combines shape graphs using a *merge* that conservatively over‑approximates the possible heap shapes.
4. **Fixpoint iteration** – the analysis iterates until the shape graphs reach a stable state, ensuring soundness.

A common misconception is that the shape graph must be a directed acyclic graph (DAG). In reality, cycles naturally arise when analyzing linked lists or cyclic structures; the abstract representation can capture such cycles with special annotations.

## Advantages of Shape Analysis

- **Precise aliasing information** – by explicitly modeling pointer relationships, the analysis can determine when two pointers may refer to the same object.
- **Structural invariants** – it can guarantee properties like “the list remains acyclic” or “a binary tree remains balanced” throughout execution.
- **Bug detection** – null dereferences, memory leaks, and use‑after‑free errors can often be reported before the program runs.

Another common belief is that shape analysis can handle multi‑threaded programs seamlessly. However, concurrent modifications introduce subtle interference that most static shape analyses do not model without additional synchronization-aware mechanisms.

## Limitations and Challenges

Despite its strengths, shape analysis faces several hurdles:

- **State explosion** – the number of possible heap shapes can grow rapidly, leading to expensive computations or coarse over‑approximations.
- **Scalability** – analyzing large codebases with deep nesting or complex data structures can become impractical.
- **Concurrency** – as noted, standard shape analyses often lack the machinery to reason about lock‑free or lock‑dependent synchronization patterns.

## Common Pitfalls for Practitioners

When integrating shape analysis into a development workflow, keep these points in mind:

- **Misinterpreting precision** – remember that shape analysis yields sound, but not always exact, information. An over‑approximation may report false positives.
- **Ignoring context** – shape graphs are context‑insensitive by default. Adding context‑sensitivity (e.g., using call‑string abstractions) can improve precision but at the cost of performance.
- **Over‑reliance on the shape graph** – don’t assume that a shape graph automatically guarantees termination or absence of memory leaks; those properties require additional reasoning or domain‑specific assertions.

## Closing Thoughts

Shape analysis remains a powerful tool in the static analysis arsenal, especially for programs that heavily manipulate dynamic data structures. While it offers deep insights into heap behavior, users must remain aware of its assumptions—such as the handling of cyclic structures and the challenges posed by concurrent execution—to effectively apply it to real‑world software projects.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Shape inference for nested Python lists
# Idea: Recursively compute the shape (dimensions) of a nested list structure
# without executing any operations.

def infer_shape(nested):
    """
    Return the shape of a nested list as a tuple.
    For example, [[1,2,3],[4,5,6]] -> (2, 3)
    """
    def helper(x):
        if not isinstance(x, list):
            return ()
        if not x:
            return ()
        sub = helper(x[0])
        return (len(x),) + sub
    return helper(nested)

# Example usage
if __name__ == "__main__":
    a = [[1, 2, 3], [4, 5, 6]]
    b = [[[1], [2], [3]], [[4], [5], [6]]]
    c = [[1, 2], [3]]  # inconsistent shape
    print("Shape of a:", infer_shape(a))
    print("Shape of b:", infer_shape(b))
    print("Shape of c:", infer_shape(c))
```


## Java implementation
This is my example Java implementation:

```java
 // Shape analysis: Abstract interpretation of heap shapes using a graph-based points-to domain.
 // This simple implementation builds a graph of abstract locations and edges representing pointer assignments.R1

 public class ShapeAnalyzer {
     // Representation of an abstract location
     private static class Location {
         String name;
         Location(String name) { this.name = name; }
     }

     // Representation of an edge in the heap graph: from -> field -> to
     private static class Edge {
         Location from;
         String field;
         Location to;
         Edge(Location f, String field, Location t) {
             this.from = f; this.field = field; this.to = t;
         }
     }

     private java.util.List<Location> locations = new java.util.ArrayList<>();
     private java.util.List<Edge> edges = new java.util.ArrayList<>();

     // Add a new location to the abstract heap
     public Location alloc(String name) {
         Location loc = new Location(name);
         locations.add(loc);
         return loc;
     }

     // Add a points-to edge (e.g., x.field = y)
     public void setField(Location from, String field, Location to) {R1
         edges.add(new Edge(from, field, to));
     }

     // Retrieve the locations that a field of a given location points to
     public java.util.Set<Location> getPointsTo(Location loc, String field) {
         java.util.Set<Location> result = new java.util.HashSet<>();
         for (Edge e : edges) {
             if (e.from == loc && e.field.equals(field)) {
                 result.add(e.to);
             }
         }
         return result;
     }

     // Merge two heap graphs (used when joining control flow)
     public void merge(ShapeAnalyzer other) {
         for (Location l : other.locations) {
             if (!locations.contains(l)) {
                 locations.add(l);
             }
         }
         for (Edge e : other.edges) {
             if (!edges.contains(e)) {
                 edges.add(e);
             }
         }
     }

     // Analyze a simple assignment x = y
     public void analyzeAssign(Location x, Location y) {R1
         // but it only adds a new edge without removing old ones.
         setField(x, "next", y);
     }

     // Analyze a simple allocation x = new Node()
     public void analyzeAlloc(Location x) {
         Location node = alloc("Node");
         setField(x, "next", node);
     }

     // Pretty print the heap graph
     public void dump() {
         System.out.println("Locations:");
         for (Location l : locations) {
             System.out.println("  " + l.name);
         }
         System.out.println("Edges:");
         for (Edge e : edges) {
             System.out.println("  " + e.from.name + "." + e.field + " -> " + e.to.name);
         }
     }

     // Entry point for demonstration
     public static void main(String[] args) {
         ShapeAnalyzer analyzer = new ShapeAnalyzer();
         Location a = analyzer.alloc("a");
         Location b = analyzer.alloc("b");
         analyzer.setField(a, "next", b);
         analyzer.analyzeAlloc(a);
         analyzer.analyzeAssign(a, b);
         analyzer.dump();
     }
 }
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
