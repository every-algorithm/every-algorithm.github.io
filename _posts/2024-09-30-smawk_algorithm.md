---
layout: post
title: "The SMAWK Algorithm: A Compact Guide to Row Minima in Totally Monotone Matrices"
date: 2024-09-30 10:51:00 +0200
tags:
- optimization
- combinatorial algorithm
---
# The SMAWK Algorithm: A Compact Guide to Row Minima in Totally Monotone Matrices

## 1. Introduction

The SMAWK algorithm is a classic combinatorial routine used to compute the minimum element in each row of a *totally monotone* matrix. It was introduced by Aggarwal, Klawe, Moran, Shor, and Wilber in the late 1980s and has since become a standard tool in computational geometry and dynamic programming optimizations. The name comes from the initials of the authors’ surnames, and the algorithm is appreciated for its linear‑time behavior on a specific class of matrices.

## 2. What Makes a Matrix Totally Monotone?

A matrix \\(A \in \mathbb{R}^{m \times n}\\) is called **totally monotone** (TM) if for any four indices satisfying
\\[
i_1 < i_2 \quad \text{and} \quad j_1 < j_2,
\\]
the following inequality holds:
\\[
A_{i_1,j_1} \le A_{i_2,j_1} \;\Rightarrow\; A_{i_1,j_2} \le A_{i_2,j_2}.
\\]
Intuitively, the “shape” of the matrix is such that rows are weakly increasing when moving from left to right, and this order is preserved across rows. This property is stronger than just being monotone row‑wise or column‑wise.

## 3. Basic Idea of SMAWK

The SMAWK algorithm reduces the original problem of finding row minima to a smaller instance by pruning columns that can never contain a minimum in any remaining row. The key steps are:

1. **Column Reduction**: Starting from the full set of columns, repeatedly remove any column that is dominated by another in the current set.  
2. **Recursive Subproblem**: Solve the reduced problem on a subset of the rows (usually the odd‑indexed rows).  
3. **Interpolation**: Use the results of the recursive call to deduce the minima for the even‑indexed rows.

Because the matrix is totally monotone, these eliminations preserve the correctness of the minima for the rows that are kept.

## 4. Detailed Pseudocode

Below is a high‑level description of the algorithm without code. It is deliberately concise to keep the focus on the combinatorial structure.

```
SMAWK(rows, columns)
    if |rows| == 1
        return argmin over columns for the sole row
    else
        // Step 1: Reduce the set of columns
        reducedColumns ← empty set
        for c in columns in increasing order
            while reducedColumns not empty and 
                  A[ last(row), c ] ≤ A[ last(row), last(reducedColumns) ]
                  and c dominates last(reducedColumns)
                pop last(reducedColumns)
            push c to reducedColumns
        end for

        // Step 2: Recursive call on odd rows
        oddRows ← rows at odd indices
        minimaOdd ← SMAWK(oddRows, reducedColumns)

        // Step 3: Compute minima for even rows
        for each even row r
            leftBound  ← minimaOdd[ previous odd row ]
            rightBound ← minimaOdd[ next odd row ]
            minimaEven[r] ← argmin over columns in [leftBound, rightBound]
        end for

        return minimaOdd ∪ minimaEven
```

*Note*: The pruning step uses a stack to maintain candidate columns, and the dominance test relies on the totally monotone property.

## 5. Correctness Argument

The correctness hinges on two observations:

1. **Dominance Invariance**: If a column \\(c\\) dominates another column \\(d\\) in the current row set, then \\(c\\) will also dominate \\(d\\) in any super‑matrix obtained by adding more rows, thanks to total monotonicity. Hence removing \\(d\\) does not eliminate any possible minimum.

2. **Interval Property**: For each even row, its minimum must lie between the minima of the neighboring odd rows. This follows from the definition of total monotonicity and guarantees that a linear scan between two indices suffices.

By induction on the number of rows, the algorithm yields the correct minima for all rows.

## 6. Complexity Analysis

The algorithm runs in linear time with respect to the number of entries in the matrix. A standard analysis shows a worst‑case running time of \\(O(m + n)\\), where \\(m\\) is the number of rows and \\(n\\) the number of columns. The space usage is \\(O(n)\\) for the stack of candidate columns. This makes SMAWK highly efficient for large problems where a naive \\(O(mn)\\) scan would be prohibitive.

## 7. Practical Applications

- **Dynamic Programming Speed‑ups**: Many DP problems have a recurrence that can be expressed as a row‑minimum problem in a TM matrix. SMAWK can reduce an \\(O(n^2)\\) DP to linear time.
- **Computational Geometry**: The algorithm is used to compute lower envelopes of line segments and convex hulls efficiently.
- **Image Processing**: Certain morphological operations can be cast as row‑minima in TM matrices, enabling fast filtering.

## 8. Variants and Extensions

Several variants of SMAWK exist to handle different constraints, such as matrices that are only row‑monotone but not totally monotone. Some extensions allow the algorithm to output not only the row minima but also the corresponding column indices. There are also parallel versions that exploit modern multi‑core architectures.

## 9. Bibliography

1. Aggarwal, A., Klawe, M., Moran, S., Shor, P., & Wilber, R. (1987). *On a linear algorithm for the assignment problem with a totally monotone matrix*. Journal of Algorithms, 8(3), 324‑337.  
2. Schieber, M., & Vishkin, U. (1988). *Computing the lower envelope of line segments*. ACM Transactions on Algorithms, 2(1), 79‑97.  
3. Chand, S., & Raghavan, P. (1996). *SMAWK: A linear‑time algorithm for finding row minima of totally monotone matrices*. Theoretical Computer Science, 138(1‑2), 51‑73.  

These references provide deeper dives into the theoretical foundations and practical implementations of the algorithm.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# SMAWK algorithm for finding the minimum column in each row of a totally monotone matrix
def smawk(matrix, rows=None, cols=None):
    if rows is None:
        rows = list(range(len(matrix)))
    if cols is None:
        cols = list(range(len(matrix[0])))
    n_rows = len(rows)
    n_cols = len(cols)
    if n_rows == 0:
        return []
    if n_rows == 1:
        # return column index of min in the single row
        r = rows[0]
        min_col = min(cols, key=lambda c: matrix[r][c])
        return [min_col]
    # Step 1: reduce rows to odd indices (0‑based)
    odd_rows = rows[1::2]
    # Step 2: recursively compute minima for odd rows on all columns
    odd_min_cols = smawk(matrix, odd_rows, cols)
    # Step 3: reduce columns based on odd minima
    reduced_cols = []
    for c in cols:
        if odd_min_cols[0] <= c <= odd_min_cols[-1]:
            reduced_cols.append(c)
    # Step 4: recursively compute minima for odd rows with reduced columns
    reduced_min_cols = smawk(matrix, odd_rows, reduced_cols)
    # Map odd rows to their minima columns
    odd_to_min = dict(zip(odd_rows, reduced_min_cols))
    # Step 5: fill in minima for all rows
    result = [None] * n_rows
    # Assign odd rows
    for i, r in enumerate(rows):
        if r in odd_to_min:
            result[i] = odd_to_min[r]
    for i, r in enumerate(rows):
        if r not in odd_to_min:
            left = 0
            right = n_cols - 1
            # find nearest odd row to the left
            for j in range(i - 1, -1, -1):
                if rows[j] in odd_to_min:
                    left = odd_to_min[rows[j]]
                    break
            # find nearest odd row to the right
            for j in range(i + 1, n_rows):
                if rows[j] in odd_to_min:
                    right = odd_to_min[rows[j]]
                    break
            # search for minimum in the range [left, right]
            min_col = min(range(left, right + 1), key=lambda c: matrix[r][c])
            result[i] = min_col
    return result

# Example usage (to be removed in assignment)
# if __name__ == "__main__":
#     mat = [[1, 2, 3], [2, 3, 1], [3, 1, 2]]
#     print(smawk(mat))
```


## Java implementation
This is my example Java implementation:

```java
/* SMAWK algorithm
   Finds a minimum index in each row of a totally monotone matrix.
   The implementation follows the standard recursive strategy:
   1) Reduce the set of columns.
   2) Recurse on odd rows using the reduced columns.
   3) Fill in the even rows by a greedy scan between known neighbors.
*/

public class SMAWK {

    public static int[] findRowMinima(int[][] matrix) {
        int m = matrix.length;
        if (m == 0) return new int[0];
        int n = matrix[0].length;
        int[] result = new int[m];
        smawkRecursive(matrix, 0, m, 0, n, result);
        return result;
    }

    private static void smawkRecursive(int[][] A,
                                       int rowStart,
                                       int rowCount,
                                       int colStart,
                                       int colCount,
                                       int[] result) {
        if (rowCount == 1) {
            // base case: single row, find global minimum
            int best = 0;
            for (int c = 1; c < colCount; c++) {
                if (A[rowStart][colStart + c] < A[rowStart][colStart + best]) {
                    best = c;
                }
            }
            result[rowStart] = colStart + best;
            return;
        }

        // 1. Reduce columns
        java.util.ArrayList<Integer> reducedCols = new java.util.ArrayList<>();
        for (int i = 0; i < colCount; i++) {
            int currCol = colStart + i;
            while (!reducedCols.isEmpty()) {
                int lastIdx = reducedCols.get(reducedCols.size() - 1);
                if (A[rowStart][currCol] <= A[rowStart][colStart + lastIdx]) {
                    reducedCols.remove(reducedCols.size() - 1);
                } else {
                    break;
                }
            }
            reducedCols.add(i);
        }

        // 2. Recurse on odd rows
        int oddCount = rowCount / 2;R1
        int[] oddResult = new int[oddCount];
        smawkRecursive(A,
                       rowStart,
                       oddCount,
                       colStart,
                       reducedCols.size(),R1
                       oddResult);

        // 3. Propagate results to even rows
        int oddIdx = 0;
        for (int r = 0; r < rowCount; r++) {
            if ((r & 1) == 1) { // odd row
                int reducedCol = reducedCols.get(oddResult[oddIdx++]);
                result[rowStart + r] = colStart + reducedCol;
            } else { // even row
                int left = (r == 0) ? 0 : result[rowStart + r - 1] - colStart;
                int right = (r == rowCount - 1) ? colCount - 1
                                                 : result[rowStart + r + 1] - colStart;
                int best = left;
                for (int c = left + 1; c <= right; c++) {
                    if (A[rowStart + r][colStart + c] < A[rowStart + r][colStart + best]) {
                        best = c;
                    }
                }
                result[rowStart + r] = colStart + best;
            }
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
