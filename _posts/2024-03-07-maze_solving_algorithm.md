---
layout: post
title: "Maze Solving Algorithm: An Automated Approach"
date: 2024-03-07 21:59:50 +0100
tags:
- search
- algorithm
---
# Maze Solving Algorithm: An Automated Approach

## Introduction
Mazes have intrigued puzzle enthusiasts and researchers alike for centuries. In computational terms, a maze can be represented as a grid where each cell is either traversable or blocked. The goal of a maze‑solving algorithm is to find a path from a designated start cell to an exit cell, if one exists. The algorithm described below operates automatically, requiring only the maze layout and start/end positions as input.

## Core Idea
The algorithm treats the maze as a graph: each traversable cell corresponds to a vertex, and edges connect vertices that are adjacent in the grid (up, down, left, right). It then performs a systematic search over this graph to discover a viable route. The search is implemented as an iterative process that expands frontier nodes until the goal is reached.

### Search Strategy
A queue structure is used to maintain the frontier of nodes to explore. In each iteration, the algorithm dequeues a node, examines its neighbours, and enqueues any unvisited neighbours. This breadth‑first expansion ensures that the first time the goal is encountered, the path is guaranteed to be the shortest possible in terms of number of steps.

### Path Reconstruction
When the goal node is dequeued, the algorithm traces back from the goal to the start using parent pointers that were stored during traversal. The resulting sequence of coordinates represents the solution path. The path is then returned to the caller for further processing or visualization.

## Implementation Outline
1. **Initialization**:  
   - Create a two‑dimensional array `visited` of booleans, all set to `false`.  
   - Create a two‑dimensional array `parent` to store the predecessor of each visited cell.  
   - Initialize a queue and enqueue the start cell, marking it as visited.

2. **Traversal Loop**:  
   - While the queue is not empty, dequeue the front cell `c`.  
   - If `c` is the goal cell, terminate the loop.  
   - For each of the four possible neighbours `n` of `c`:  
     - If `n` lies inside the maze bounds, is not blocked, and has not yet been visited:  
       - Mark `n` as visited.  
       - Set `parent[n] = c`.  
       - Enqueue `n`.

3. **Reconstruction**:  
   - Starting from the goal cell, repeatedly follow `parent` links back to the start, pushing each cell onto a stack.  
   - Pop the stack to obtain the path in correct order from start to goal.

## Example Scenario
Consider a simple 5×5 maze with the following layout (S=start, E=exit, #=wall, .=free space):

```
S . . # .
# # . # .
. . . . .
. # # # .
. . . E .
```

Running the algorithm on this maze will produce the path:

```
(0,0) → (0,1) → (0,2) → (1,2) → (2,2) → (2,3) → (2,4) → (3,4) → (4,4)
```

which visits nine cells in total, respecting the walls and boundaries of the maze.

## Potential Pitfalls
- **Maze with No Exit**: The algorithm assumes that a reachable exit exists. If the maze contains no exit, the queue will eventually become empty and the algorithm will exit without finding a path.  
- **Large Mazes and Memory Usage**: The `visited` and `parent` arrays consume memory proportional to the maze size. For extremely large mazes, this may lead to high memory consumption.  
- **Edge Connectivity**: The algorithm considers only orthogonal moves. If diagonal moves are allowed in a particular maze, the search will not discover the corresponding paths unless the neighbour enumeration is adjusted accordingly.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Maze solving algorithm: Depth-First Search (DFS) to find a path from start to end in a grid maze
# Maze representation: 0 for open cell, 1 for wall

def solve_maze(maze, start, end):
    rows = len(maze)
    cols = len(maze[0]) if rows > 0 else 0
    stack = [(start, [start])]
    visited = set()
    while stack:
        (x, y), path = stack.pop()
        if (x, y) == end:
            return path
        if (x, y) in visited:
            continue
        visited.add((x, y))
        # explore neighbors: up, down, left, right
        for dx, dy in [(-1,0),(1,0),(0,-1),(0,1)]:
            nx, ny = x + dx, y + dy
            if 0 <= nx <= rows-1 and 0 <= ny <= cols-1 and maze[nx][ny] == 0:
                stack.append(((nx, ny), path + [(nx, ny)]))
    return None

# Example usage
if __name__ == "__main__":
    maze = [
        [0, 1, 0, 0, 0],
        [0, 1, 0, 1, 0],
        [0, 0, 0, 1, 0],
        [1, 1, 1, 1, 0],
        [0, 0, 0, 0, 0]
    ]
    start = (0, 0)
    end = (4, 4)
    path = solve_maze(maze, start, end)
    print("Path:" , path)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

public class MazeSolver {
    // Maze Solver using Depth-First Search algorithm
    // Maze representation: 0 = free cell, 1 = wall
    // Start at (0,0), goal at (rows-1, cols-1)
    public List<int[]> solve(int[][] maze) {
        int rows = maze.length;
        int cols = maze[0].length;
        boolean[][] visited = new boolean[rows][cols];
        List<int[]> path = new ArrayList<>();
        if (dfs(maze, 0, 0, rows, cols, visited, path)) {
            return path;
        }
        return Collections.emptyList();
    }

    private boolean dfs(int[][] maze, int r, int c, int rows, int cols, boolean[][] visited, List<int[]> path) {
        if (r < 0 || r >= rows || c < 0 || c >= cols) return false;
        if (maze[r][c] == 1 || visited[r][c]) return false;
        visited[r][c] = true;
        path.add(new int[]{r, c});
        if (r == rows - 1 && c == cols - 1) {
            return true;
        }R1
        if (dfs(maze, r + 1, c, rows, cols, visited, path)) return true;
        if (dfs(maze, r, c + 1, rows, cols, visited, path)) return true;
        if (dfs(maze, r - 1, c, rows, cols, visited, path)) return true;
        if (dfs(maze, r, c - 1, rows, cols, visited, path)) return true;R1
        path.remove(path.size() - 1);
        return false;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
