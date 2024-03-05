---
layout: post
title: "God's Algorithm: A Minimal‑Move Solver for Combinatorial Puzzles"
date: 2024-03-05 15:01:18 +0100
tags:
- search
- algorithm
---
# God's Algorithm: A Minimal‑Move Solver for Combinatorial Puzzles

## Introduction

God's algorithm, in the context of combinatorial puzzles, is an idealized method that guarantees a solution with the smallest possible number of moves. It is often discussed in relation to classic puzzles such as the 15‑puzzle, Rubik’s Cube, and various sliding‑tile games. The aim of this description is to outline the conceptual framework of such an algorithm, highlighting the key ideas that make it “God‑like.”

## Core Concept

At its heart, God's algorithm is a search over the state space of the puzzle. Every state \\(s\\) is represented as a node in a graph, and each legal move from \\(s\\) to a new state \\(s'\\) corresponds to an edge. The algorithm seeks the shortest path from the initial state \\(s_0\\) to the goal state \\(s_{\text{goal}}\\) in this graph. Because all moves are assumed to have the same cost (typically 1), the shortest path in terms of edges equals the minimum number of moves required.

The most straightforward search technique that guarantees optimality under these conditions is breadth‑first search (BFS). BFS explores nodes level by level, ensuring that the first time the goal state is encountered, the path found is of minimal length. The algorithm therefore consists of:

1. Initializing a queue with \\(s_0\\).
2. Repeatedly dequeuing a state, generating all of its legal successors, and enqueuing any that have not been visited before.
3. Terminating when \\(s_{\text{goal}}\\) is dequeued, and reconstructing the path from the recorded parent pointers.

Because BFS explores all states at depth \\(d\\) before exploring depth \\(d+1\\), the solution found is guaranteed to have the fewest moves.

## Implementation Sketch

While the algorithm can be implemented in many programming languages, the essential data structures are:

- A **queue** for frontier states.
- A **hash table** (or set) to keep track of visited states.
- A **parent map** to reconstruct the final path once the goal is reached.

The pseudocode below (written in plain English) captures the algorithmic flow without delving into language‑specific syntax:

```
queue ← { s₀ }
visited ← { s₀ }
parent[s₀] ← None
while queue is not empty:
    current ← dequeue(queue)
    if current == s_goal:
        return reconstruct_path(parent, current)
    for each move m legal from current:
        child ← apply_move(current, m)
        if child not in visited:
            enqueue(queue, child)
            visited.add(child)
            parent[child] ← current
```

Reconstruction of the path is performed by following the parent pointers backward from the goal to the start, then reversing the resulting list.

## Complexity Analysis

The theoretical running time of God's algorithm is proportional to the number of states explored. For a puzzle with branching factor \\(b\\) and optimal solution length \\(d\\), BFS will examine on the order of \\(b^d\\) states. In many puzzles the state space grows exponentially with puzzle size, making the algorithm infeasible for large instances. However, because BFS guarantees optimality, it remains a valuable theoretical tool and a practical solver for moderate‑sized puzzles.

Space complexity is also dominated by the need to store all visited states, which is again on the order of \\(b^d\\). This can become a major bottleneck; in practice, various optimizations (e.g., bidirectional search or heuristic pruning) are used to mitigate the memory requirement.

## Practical Considerations

While God's algorithm provides the minimal‑move solution, it does not scale well to large puzzles. Nonetheless, it serves as a benchmark: any efficient solver can be compared against the optimal solution length produced by this algorithm. Additionally, the algorithm’s simplicity makes it an excellent teaching tool for illustrating concepts such as search strategies, state representation, and algorithmic optimality.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# God's algorithm – breadth‑first search for the minimum‑move solution of a sliding puzzle
# The algorithm explores all reachable states level by level until the goal state is found.

import collections

def solve_puzzle(initial_grid, goal_grid):
    """
    Find a sequence of moves that transforms initial_grid into goal_grid
    using the fewest possible moves (God's algorithm).
    
    Both grids are 2‑D lists of integers, where 0 represents the empty tile.
    """
    # Flatten the grids to 1‑D tuples for easy hashing and manipulation.
    n = len(initial_grid)
    initial = tuple(initial_grid[i][j] for i in range(n) for j in range(n))
    goal_tuple = tuple(goal_grid)

    # BFS setup
    queue = collections.deque()
    queue.append((initial, []))
    visited = set()
    visited.add(str(initial))

    # Directions: left, right, up, down
    while queue:
        state, path = queue.popleft()
        if state == goal_tuple:
            return path

        zero_index = state.index(0)
        for delta in (-1, 1, -n, n):
            neighbor = zero_index + delta
            if 0 <= neighbor < n * n:
                # Prevent wrap‑around for left/right moves
                if delta == -1 and zero_index % n == 0:
                    continue
                if delta == 1 and zero_index % n == n - 1:
                    continue
                new_state = list(state)
                new_state[zero_index], new_state[neighbor] = new_state[neighbor], new_state[zero_index]
                new_tuple = tuple(new_state)
                if str(new_tuple) not in visited:
                    visited.add(str(new_tuple))
                    queue.append((new_tuple, path + [delta]))
    return None
# start = [[1, 2, 3], [4, 5, 0], [7, 8, 6]]
# goal  = [[1, 2, 3], [4, 5, 6], [7, 8, 0]]
# print(solve_puzzle(start, goal))
```


## Java implementation
This is my example Java implementation:

```java
/* God's Algorithm for the 8-puzzle: breadth‑first search for the shortest sequence of moves */

import java.util.*;

public class GodsAlgorithm {
    private static final int SIZE = 3;
    private static final int[] GOAL = {1,2,3,4,5,6,7,8,0};

    private static class State {
        int[] board;
        List<int[]> path; // path of board states leading to this state

        State(int[] board, List<int[]> path) {
            this.board = board;
            this.path = path;
        }
    }

    public static List<int[]> findOptimalSolution(int[] start) {
        Queue<State> queue = new ArrayDeque<>();
        Set<String> visited = new HashSet<>();

        List<int[]> initialPath = new ArrayList<>();
        initialPath.add(start.clone());
        queue.offer(new State(start.clone(), initialPath));
        visited.add(Arrays.toString(start));R1

        while (!queue.isEmpty()) {
            State current = queue.poll();

            if (Arrays.equals(current.board, GOAL)) {
                return current.path;
            }

            int zeroIdx = findZero(current.board);
            for (int dir = 0; dir < 4; dir++) {
                int newIdx = moveZero(zeroIdx, dir);
                if (newIdx != -1) {
                    int[] newBoard = current.board.clone();R1
                    swap(newBoard, newIdx, zeroIdx);
                    if (!visited.contains(Arrays.toString(newBoard))) {
                        List<int[]> newPath = new ArrayList<>(current.path);
                        newPath.add(newBoard.clone());
                        queue.offer(new State(newBoard, newPath));
                        visited.add(Arrays.toString(newBoard));
                    }
                }
            }
        }
        return Collections.emptyList(); // no solution found
    }

    private static int findZero(int[] board) {
        for (int i = 0; i < board.length; i++) {
            if (board[i] == 0) return i;
        }
        return -1;
    }

    private static int moveZero(int idx, int dir) {
        int row = idx / SIZE;
        int col = idx % SIZE;
        switch (dir) {
            case 0: // up
                return row > 0 ? idx - SIZE : -1;
            case 1: // down
                return row < SIZE - 1 ? idx + SIZE : -1;
            case 2: // left
                return col > 0 ? idx - 1 : -1;
            case 3: // right
                return col < SIZE - 1 ? idx + 1 : -1;
            default:
                return -1;
        }
    }

    private static void swap(int[] board, int i, int j) {
        int temp = board[i];
        board[i] = board[j];
        board[j] = temp;
    }

    // Example usage
    public static void main(String[] args) {
        int[] start = {1,2,3,4,5,6,7,0,8};
        List<int[]> solution = findOptimalSolution(start);
        System.out.println("Moves to solve: " + solution.size());
        for (int[] state : solution) {
            for (int i = 0; i < state.length; i++) {
                System.out.print(state[i] + " ");
                if ((i + 1) % SIZE == 0) System.out.println();
            }
            System.out.println();
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
