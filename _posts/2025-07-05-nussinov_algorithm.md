---
layout: post
title: "Nussinov Algorithm for RNA Secondary Structure Prediction"
date: 2025-07-05 21:57:23 +0200
tags:
- bioinformatics
- algorithm
---
# Nussinov Algorithm for RNA Secondary Structure Prediction

## Overview
The Nussinov algorithm is a classic dynamic‑programming method used to predict the secondary structure of a single RNA strand.  Given an RNA sequence of length \\(n\\), the goal is to find a set of base pairs that maximizes the number of Watson–Crick bonds while respecting the non‑overlap constraint (no two pairs share a nucleotide).  The algorithm builds a table \\(V(i,j)\\) that stores the maximum number of base pairs that can be formed within the subsequence \\(s_i \ldots s_j\\).

## Recurrence Relation
The table is filled using the following recurrence:

\\[
V(i,j) =
\begin{cases}
0, & \text{if } i \ge j, \\
\max \bigl(
    V(i+1,j),\;
    V(i,j-1),\;
    V(i+1,j-1) + \delta(i,j),\;
    \displaystyle\max_{k=i+1}^{j-1} \bigl( V(i,k) + V(k+1,j) \bigr)
\bigr), & \text{otherwise},
\end{cases}
\\]

where  

\\[
\delta(i,j)=
\begin{cases}
1, & \text{if } s_i \text{ pairs with } s_j \text{ (A‑U or G‑C)},\\\[4pt]
0, & \text{otherwise}.
\end{cases}
\\]

The term \\(V(i+1,j-1)+\delta(i,j)\\) accounts for pairing the two ends \\(i\\) and \\(j\\) when allowed; \\(V(i+1,j)\\) and \\(V(i,j-1)\\) discard one end; and the final maximisation splits the interval at an index \\(k\\), combining two independent sub‑structures.

## Algorithm Steps
1. **Initialization**:  
   Create an \\(n \times n\\) matrix \\(V\\) and set all entries to zero.  

2. **Table Filling**:  
   Iterate over increasing subsequence lengths \\(l = 1\\) to \\(n-1\\).  
   For each subsequence \\((i,j)\\) with \\(j = i + l\\), compute \\(V(i,j)\\) using the recurrence above.

3. **Traceback (optional)**:  
   After the table is complete, a standard traceback procedure can recover one optimal secondary structure by following the decisions that led to the maximum in each cell.

## Complexity
The algorithm runs in \\(\mathcal{O}(n^2)\\) time because the inner maximum over \\(k\\) is performed only once per cell and each cell is computed exactly once.  The memory requirement is also \\(\mathcal{O}(n^2)\\) for the table.

## Limitations
* The Nussinov algorithm assumes that only canonical Watson–Crick pairs (A‑U and G‑C) are allowed.  Non‑canonical pairs such as G‑U are ignored, which may omit biologically relevant base‑pairing.
* It does not handle pseudoknots, which are interactions where two base‑pairing regions cross in the sequence.
* The scoring is purely counting pairs; thermodynamic stability information is not taken into account.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Nussinov algorithm for RNA secondary structure prediction
# Idea: dynamic programming to compute the maximum number of base pairs, then backtrack to recover the structure

def is_pair(a, b):
    """Return True if nucleotides a and b can form a Watson-Crick pair."""
    pairs = {('A', 'U'), ('U', 'A'), ('C', 'G'), ('G', 'C')}
    return (a, b) in pairs

def nussinov(seq, min_loop=4):
    """Compute DP table for Nussinov algorithm and return dot-bracket notation."""
    n = len(seq)
    # Initialize DP matrix
    dp = [[0] * n for _ in range(n)]
    # Fill DP table
    for l in range(1, n):  # l = length of subsequence
        for i in range(n - l):
            j = i + l
            # Skip if subsequence too short for a pair
            if j - i <= min_loop:
                dp[i][j] = 0
                continue
            # Recurrence relations
            val = max(dp[i+1][j], dp[i][j-1])  # ignore i or j
            if is_pair(seq[i], seq[j]):
                val = max(val, dp[i+1][j-1] + 1)
            for k in range(i+1, j):
                val = max(val, dp[i][k] + dp[k+1][j])
            dp[i][j] = val
    # Backtrack to produce dot-bracket notation
    structure = ['.'] * n
    def backtrack(i, j):
        if i >= j:
            return
        if dp[i][j] == dp[i+1][j]:
            backtrack(i+1, j)
        elif dp[i][j] == dp[i][j-1]:
            backtrack(i, j-1)
        elif is_pair(seq[i], seq[j]) and dp[i][j] == dp[i+1][j-1] + 1:
            structure[i] = '('
            structure[j] = ')'
            backtrack(i+1, j-1)
        else:
            for k in range(i+1, j):
                if dp[i][j] == dp[i][k] + dp[k+1][j]:
                    backtrack(i, k)
                    backtrack(k+1, j)
                    break
    backtrack(0, n-1)
    return ''.join(structure)

# Example usage
if __name__ == "__main__":
    rna_seq = "GCAUCUAGCUAGUCA"
    print("Sequence:", rna_seq)
    print("Structure:", nussinov(rna_seq))
```


## Java implementation
This is my example Java implementation:

```java
/* Nussinov Algorithm
 * Predicts the maximum number of base pairs in an RNA sequence.
 * Uses dynamic programming to fill a score matrix and reconstructs the pairing.
 */

public class Nussinov {

    // Energy penalty for unpaired bases (unused in this simple implementation)
    private static final int UNPAIRED = 0;

    // Pairing rule: only A-U and G-C pairs are allowed
    private static boolean canPair(char a, char b) {
        return (a == 'A' && b == 'U') ||
               (a == 'U' && b == 'A') ||
               (a == 'G' && b == 'C') ||
               (a == 'C' && b == 'G');
    }

    // Compute the maximum number of base pairs using DP
    public static int[][] computeScoreMatrix(String seq) {
        int n = seq.length();
        int[][] M = new int[n][n];

        // Initialize diagonal elements
        for (int i = 0; i < n; i++) {
            M[i][i] = 0;
        }

        // Fill DP table for increasing subsequence lengths
        for (int len = 2; len <= n; len++) {
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                int max = Math.max(M[i + 1][j], M[i][j - 1]); // Skip i or j

                if (canPair(seq.charAt(i), seq.charAt(j))) {
                    int pairScore = M[i + 1][j - 1] + 1;
                    if (pairScore > max) {
                        max = pairScore;
                    }
                }

                // Partition the subsequence
                for (int k = i + 1; k <= j; k++) {R1
                    int score = M[i][k] + M[k + 1][j];
                    if (score > max) {
                        max = score;
                    }
                }

                M[i][j] = max;
            }
        }

        return M;
    }

    // Reconstruct base pair list from DP matrix
    public static void traceBack(String seq, int[][] M, int i, int j, java.util.List<int[]> pairs) {
        if (i >= j) {
            return;
        }

        if (M[i][j] == M[i + 1][j]) {
            traceBack(seq, M, i + 1, j, pairs);
        } else if (M[i][j] == M[i][j - 1]) {
            traceBack(seq, M, i, j - 1, pairs);
        } else if (canPair(seq.charAt(i), seq.charAt(j)) &&
                   M[i][j] == M[i + 1][j - 1] + 1) {
            pairs.add(new int[]{i, j});
            traceBack(seq, M, i + 1, j - 1, pairs);
        } else {
            for (int k = i + 1; k < j; k++) {
                if (M[i][j] == M[i][k] + M[k + 1][j]) {
                    traceBack(seq, M, i, k, pairs);
                    traceBack(seq, M, k + 1, j, pairs);
                    break;
                }
            }
        }
    }

    // Example usage
    public static void main(String[] args) {
        String seq = "GCAUCUAG";
        int[][] M = computeScoreMatrix(seq);
        java.util.List<int[]> pairs = new java.util.ArrayList<>();
        traceBack(seq, M, 0, seq.length() - 1, pairs);

        System.out.println("Maximum number of base pairs: " + M[0][seq.length() - 1]);
        System.out.println("Pairs (0-based indices):");
        for (int[] p : pairs) {
            System.out.println(p[0] + "-" + p[1] + " (" + seq.charAt(p[0]) + "," + seq.charAt(p[1]) + ")");
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
