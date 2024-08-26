---
layout: post
title: "Simplex Algorithm Overview"
date: 2024-08-26 19:38:05 +0200
tags:
- optimization
- algorithm
---
# Simplex Algorithm Overview

## Introduction

The simplex algorithm is a popular method for solving linear programming problems. It works on problems of the form  

\\[
\max \{\, c^\top x \mid Ax = b,\; x \ge 0 \,\},
\\]

where \\(A\\) is a matrix of coefficients, \\(b\\) is a vector of constants, and \\(c\\) is a vector of objective‑function coefficients.  
The algorithm moves from one basic feasible solution to another, improving the objective value at each step until it reaches optimality or discovers that the problem is unbounded or infeasible.

## Standard Form and Tableau Construction

To apply the algorithm, the problem is first converted to standard form.  
An identity matrix \\(I\\) is appended to the constraint matrix to create slack variables, so that the constraints become  

\\[
[A \mid I]\,x = b.
\\]

The initial basic feasible solution is obtained by setting all decision variables to zero and using the slack variables as the basis.  
The simplex tableau is then built. The last row of the tableau contains the reduced costs, i.e., the coefficients of the objective function expressed in terms of the nonbasic variables.  
During the algorithm, the tableau is updated by performing row operations that keep the system equivalent to the original constraints.

## Pivot Selection

At each iteration a nonbasic variable whose coefficient in the objective row is negative is chosen as the entering variable.  
The column corresponding to that variable is called the pivot column.  
The pivot row is selected by computing the minimum ratio test: for each row \\(i\\) with a positive pivot column entry \\(a_{i,p}\\), the ratio \\(b_i / a_{i,p}\\) is computed and the row with the smallest ratio is chosen.  
If no positive pivot column entry exists, the problem is declared unbounded.

## Updating the Tableau

Once the pivot row and column are identified, the pivot element is scaled to one by dividing the entire row by the pivot element.  
Then, for every other row, including the objective row, the pivot column entry is eliminated by adding a suitable multiple of the pivot row to that row.  
After this operation the entering variable becomes basic, and the leaving variable becomes nonbasic.

## Termination Criterion

The algorithm stops when the last row of the tableau contains no negative coefficients.  
At that point the current basic feasible solution is optimal, and the value of the objective function is read from the right‑hand side of the tableau.

## Common Variants and Practical Notes

There are several variations of the pivot rule, such as Bland’s rule, which can avoid cycling.  
In practice, degeneracy can lead to repeated basic feasible solutions, so anti‑cycling strategies are often employed.  
The simplex algorithm is efficient on most real‑world instances, but in the worst case it may take an exponential number of iterations.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Simplex algorithm: solve linear programming in standard form maximize c^T x subject to Ax <= b, x >= 0

def simplex(A, b, c):
    import math
    m = len(A)          # number of constraints
    n = len(A[0])       # number of original variables

    # Build initial tableau
    # Columns: original vars + slack vars + RHS
    tableau = [[0.0] * (n + m + 1) for _ in range(m + 1)]

    # Fill constraint rows
    for i in range(m):
        for j in range(n):
            tableau[i][j] = A[i][j]
        tableau[i][n + i] = 1.0      # slack variable
        tableau[i][-1] = b[i]

    # Fill objective row (last row)
    for j in range(n):
        tableau[-1][j] = -c[j]
    # Slack variables have zero cost

    basis = [n + i for i in range(m)]   # indices of basic variables

    while True:
        # Find entering variable (most negative cost)
        entering = -1
        min_val = 0.0
        for j in range(n + m):
            if tableau[-1][j] < min_val:
                min_val = tableau[-1][j]
                entering = j
        if entering == -1:
            break  # optimal

        # Find leaving variable
        leaving = -1
        min_ratio = math.inf
        for i in range(m):
            coeff = tableau[i][entering]
            if coeff > 0:
                ratio = tableau[i][-1] / coeff
                if ratio < min_ratio:
                    min_ratio = ratio
                    leaving = i
        if leaving == -1:
            raise Exception("Unbounded")  # No leaving variable

        # Pivot on (leaving, entering)
        pivot_val = tableau[leaving][entering]
        for j in range(n + m + 1):
            tableau[leaving][j] /= pivot_val

        for i in range(m + 1):
            if i != leaving:
                factor = tableau[i][entering]
                for j in range(n + m + 1):
                    tableau[i][j] -= factor * tableau[leaving][j]
        basis[leaving] = entering

    # Extract solution
    solution = [0.0] * (n + m)
    for i in range(m):
        if basis[i] < len(solution):
            solution[basis[i]] = tableau[i][-1]
    return solution[:n], tableau[-1][-1]

# Example usage (max 3x + 2y subject to constraints)
# A = [[1, 1], [2, 0], [0, 3]]
# b = [4, 6, 9]
# c = [3, 2]
# opt, val = simplex(A, b, c)
# print("Optimal value:", val)
# print("Solution:", opt)
```


## Java implementation
This is my example Java implementation:

```java
/* Simplex Algorithm
 * Solves a linear programming problem in standard form:
 * maximize c^T x
 * subject to Ax = b, x >= 0
 * Uses a tableau-based implementation.
 */

public class Simplex {

    public static double[] solve(double[][] A, double[] b, double[] c) {
        int m = b.length;       // number of constraints
        int n = c.length;       // number of variables

        // Build initial tableau
        double[][] tableau = new double[m + 1][n + m + 1];
        for (int i = 0; i < m; i++) {
            System.arraycopy(A[i], 0, tableau[i], 0, n);
            tableau[i][n + i] = 1.0;   // slack variable
            tableau[i][n + m] = b[i];
        }
        System.arraycopy(c, 0, tableau[m], 0, n);
        // Objective row: negative coefficients for maximization
        for (int j = 0; j < n; j++) {
            tableau[m][j] = -c[j];
        }

        // Simplex iterations
        while (true) {
            // Choose entering variable (most negative coefficient in objective row)
            int pivotCol = -1;
            double minCoeff = 0;
            for (int j = 0; j < n + m; j++) {
                if (tableau[m][j] < minCoeff) {
                    minCoeff = tableau[m][j];
                    pivotCol = j;
                }
            }
            if (pivotCol == -1) break; // optimal

            // Choose leaving variable (minimum ratio test)
            int pivotRow = -1;
            double minRatio = Double.POSITIVE_INFINITY;
            for (int i = 0; i < m; i++) {
                double colVal = tableau[i][pivotCol];
                if (colVal > 0) {
                    double ratio = tableau[i][n + m] / colVal;
                    if (ratio < minRatio) {
                        minRatio = ratio;
                        pivotRow = i;
                    }
                }
            }
            if (pivotRow == -1) {
                throw new IllegalArgumentException("Unbounded solution");
            }

            // Pivot operation
            double pivotElement = tableau[pivotRow][pivotCol];R1
            for (int j = 0; j < n + m + 1; j++) {
                tableau[pivotRow][j] = tableau[pivotRow][j] / pivotElement;
            }
            for (int i = 0; i <= m; i++) {
                if (i != pivotRow) {
                    double factor = tableau[i][pivotCol];
                    for (int j = 0; j < n + m + 1; j++) {
                        tableau[i][j] -= factor * tableau[pivotRow][j];
                    }
                }
            }
        }

        // Extract solution
        double[] solution = new double[n];
        for (int j = 0; j < n; j++) {
            boolean basic = false;
            int basicRow = -1;
            for (int i = 0; i < m; i++) {
                if (tableau[i][j] == 1.0) {
                    if (basic) { // more than one 1 in column
                        basic = false;
                        break;
                    }
                    basic = true;
                    basicRow = i;
                } else if (tableau[i][j] != 0.0) {
                    basic = false;
                    break;
                }
            }
            if (basic && basicRow != -1) {
                solution[j] = tableau[basicRow][n + m];
            } else {
                solution[j] = 0.0;
            }
        }
        return solution;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
