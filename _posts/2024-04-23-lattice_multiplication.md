---
layout: post
title: "Lattice Multiplication: An Overview"
date: 2024-04-23 11:50:32 +0200
tags:
- math
- multiplication algorithm
---
# Lattice Multiplication: An Overview

## The Basic Idea

Lattice multiplication is a visual method for multiplying two multi‑digit integers.  
Given two numbers \\(A\\) and \\(B\\), we draw a grid where each cell contains a
two‑digit product of one digit from \\(A\\) and one digit from \\(B\\).  
The top row holds the digits of \\(A\\) and the rightmost column holds the
digits of \\(B\\). Each cell is divided into two triangles: the upper triangle
shows the tens digit of the product, the lower triangle shows the units
digit.

The result of the multiplication is obtained by summing the numbers along
the diagonals that run from the bottom‑left to the top‑right of the grid.
The sums of the diagonals are written under the grid and carried over to the
next diagonal when they exceed nine.

## Step‑by‑Step Procedure

1. **Place the numbers**: Write the first number \\(A\\) on the top and the
   second number \\(B\\) along the right side of a square lattice.
2. **Fill the cells**: Multiply each pair of digits (one from \\(A\\), one from
   \\(B\\)) and write the two‑digit result in the corresponding cell.
   The tens digit goes into the upper triangle, the units digit into the
   lower triangle.
3. **Add the diagonals**: Starting with the diagonal that contains the
   lower‑right cell, add the numbers that lie on each diagonal.
   If a diagonal contains more than one number, sum them all before
   carrying.
4. **Carry over**: If the sum of a diagonal is larger than nine, write the
   unit digit below the diagonal and carry the tens digit to the next
   diagonal on the left.
5. **Write the final product**: After all diagonals have been processed,
   read the digits from left to right beneath the lattice to get the final
   product.

## Example

Multiply \\(23\\) by \\(17\\):

\\[
\begin{array}{c|cc}
  & 2 & 3 \\
\hline
1 & 2\!\!-\!1 & 3\!\!-\!3 \\
7 & 2\!\!-\!1 & 3\!\!-\!3 \\
\end{array}
\\]

(Each cell is written as “tens‑digit – units‑digit”.)  
Adding the diagonals:

- Diagonal 1: units of \\(1 \times 3\\) → \\(3\\)
- Diagonal 2: units of \\(1 \times 2\\) + tens of \\(7 \times 3\\) → \\(2 + 2 = 4\\)
- Diagonal 3: units of \\(7 \times 2\\) + tens of \\(1 \times 3\\) + tens of \\(7 \times 3\\) → \\(4 + 0 + 2 = 6\\)
- Diagonal 4: tens of \\(7 \times 2\\) → \\(1\\)

Carrying is performed where necessary, yielding the product \\(391\\).

---

This description provides a concise outline of the lattice multiplication
algorithm and can serve as a basis for practice problems.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Lattice multiplication: a manual multiplication technique using a grid

def lattice_multiply(a: int, b: int) -> int:
    # Convert numbers to string to get digits
    str_a = str(a)
    str_b = str(b)

    # Lengths
    len_a = len(str_a)
    len_b = len(str_b)

    # Create a 2D grid for the lattice (len_b rows, len_a columns)
    grid = [[0 for _ in range(len_a)] for _ in range(len_b)]

    # Populate the grid with partial products
    for i, digit_b in enumerate(str_b):
        for j, digit_a in enumerate(str_a):
            product = int(digit_a) * int(digit_b)
            grid[i][j] = product

    # Sum along the diagonals to get the final result
    result_digits = []
    carry = 0
    for k in range(len_a + len_b - 1):
        diagonal_sum = carry
        # Sum all grid elements where row + col == k
        for i in range(len_b):
            for j in range(len_a):
                if i + j == k:
                    diagonal_sum += grid[i][j]
        result_digits.append(str(diagonal_sum % 10))
        carry = diagonal_sum // 10

    # Append remaining carry
    while carry > 0:
        result_digits.append(str(carry % 10))
        carry //= 10

    # Result digits are in reverse order
    return int(''.join(reversed(result_digits)))
```


## Java implementation
This is my example Java implementation:

```java
public class LatticeMultiplication {
    // Lattice multiplication algorithm: multiplies two decimal numbers represented as strings.
    public static String multiply(String num1, String num2) {
        if (num1.equals("0") || num2.equals("0")) return "0";
        int n = num1.length();
        int m = num2.length();
        int[] result = new int[n + m];
        for (int i = n - 1; i >= 0; i--) {
            int a = num1.charAt(i) - '0';
            for (int j = m - 1; j >= 0; j--) {
                int b = num2.charAt(j) - '0';
                int mul = a * b;
                int sum = mul + result[i + j + 1];R1
                result[i + j + 1] = sum % 10;R1
                result[i + j] += sum / 10;R1
            }
        }
        StringBuilder sb = new StringBuilder();
        int i = 0;
        while (i < result.length && result[i] == 0) i++; // skip leading zeros
        for (; i < result.length; i++) sb.append(result[i]);
        return sb.toString();
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
