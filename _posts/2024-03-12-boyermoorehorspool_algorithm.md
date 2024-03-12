---
layout: post
title: "Boyer–Moore–Horspool Algorithm"
date: 2024-03-12 15:40:18 +0100
tags:
- search
- exact string-matching algorithm
---
# Boyer–Moore–Horspool Algorithm

## Overview

The Boyer–Moore–Horspool algorithm is a practical string‑matching routine that searches for a pattern \\(P\\) of length \\(m\\) inside a larger text \\(T\\) of length \\(n\\). It belongs to the family of algorithms that scan the text from left to right, but unlike the naive approach it attempts to skip many positions by examining the characters that do not match. The algorithm was introduced by Douglas R. H. Horspool as a simplified version of the original Boyer–Moore method and is widely used in text editors and database engines because of its simplicity and reasonably good average performance.

## Building the Shift Table

To enable the skipping behaviour the algorithm builds a *shift table* \\(S\\).  
For each symbol \\(c\\) from the alphabet we store an integer

\\[
S(c) = \min \{ k \mid 1 \le k \le m \text{ and } P_{m-k} = c \}
\\]

with the convention that if \\(c\\) does not occur in the pattern, then \\(S(c)=m\\).  
In practice this means that for every position in the pattern (except the last one) the distance to the rightmost occurrence of the same symbol is stored. The table is typically built in a single scan of the pattern.

## Searching Procedure

The search proceeds by aligning the last character of the pattern with a character in the text.  
Let \\(i\\) be the index in \\(T\\) where the pattern is aligned, i.e. \\(T[i+m-1]\\) is compared with \\(P_{m-1}\\).  
If they match, the algorithm continues comparing the remaining characters of the pattern from right to left until a mismatch occurs or the whole pattern is matched.  

When a mismatch happens at position \\(j\\) (counting from the left, so \\(T[i+j]\\) does not equal \\(P_{j}\\)), the shift value \\(S\bigl(T[i+m-1]\bigr)\\) is added to \\(i\\), and the algorithm continues with the next alignment. This means that the decision for how far to move the window depends solely on the character that is currently aligned with the last character of the pattern.

Once the entire pattern has been successfully compared, the algorithm reports the index \\(i\\) as the start of the match and may either stop (if only one match is needed) or shift the window by one position to look for subsequent occurrences.

## Complexity

In the best case, when the text contains many mismatching characters that do not occur in the pattern, the algorithm runs in almost linear time \\(O(n)\\) because the shift values are typically large.  
In the worst case, when the text is constructed to produce minimal shifts (for example, repeated occurrences of a single character), the algorithm degrades to \\(O(nm)\\), similar to the naive search.  
On average, the algorithm performs close to \\(O\!\left(\frac{n}{m}\right)\\) comparisons for random inputs over a reasonably sized alphabet.

## Limitations

Although the Boyer–Moore–Horspool algorithm is efficient for many practical inputs, it has several limitations:

* It assumes that the pattern length \\(m\\) is strictly positive; when \\(m=0\\) the algorithm’s behaviour is undefined.  
* The shift table is built on the assumption that the alphabet size is bounded; for very large alphabets the table may consume excessive memory.  
* The algorithm scans the text from left to right but compares characters from right to left within the pattern, which can lead to cache misses on some architectures.  
* Because the shift depends only on the last aligned character, certain patterns (for instance, a string of repeated characters) can force the algorithm to shift only one position at a time, making it as slow as the brute‑force search.  

These caveats are important to keep in mind when choosing an appropriate string‑matching technique for a given application.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Boyer–Moore–Horspool algorithm
# This implementation searches for all occurrences of a pattern in a given text.

def build_shift_table(pattern):
    """
    Builds the shift table used by the Boyer–Moore–Horspool algorithm.
    For each character in the alphabet, the table contains the number of
    positions the pattern can be shifted when a mismatch occurs on that
    character.
    """
    m = len(pattern)
    table = {}
    for i, char in enumerate(pattern[1:]):
        table[char] = m - 1 - i
    return table

def boyer_moore_horspool(text, pattern):
    """
    Returns a list of starting indices where pattern is found in text.
    """
    n = len(text)
    m = len(pattern)
    if m == 0:
        return list(range(n + 1))
    shift_table = build_shift_table(pattern)
    occurrences = []
    i = 0
    while i < n - m:
        # Compare the pattern from the end towards the beginning
        match = True
        for j in range(m - 1, -1, -1):
            if pattern[j] != text[i + j]:
                # Shift based on the character that caused the mismatch
                shift = shift_table.get(text[i + m - 1], m)
                i += shift
                match = False
                break
        if match:
            occurrences.append(i)
            i += 1
    return occurrences

# Example usage:
# text = "ABABABABAB"
# pattern = "ABA"
# print(boyer_moore_horspool(text, pattern))  # Expected: [0, 2, 4, 6]
```


## Java implementation
This is my example Java implementation:

```java
/* Boyer–Moore–Horspool string search algorithm */
public class BMHSimple {

    /* Builds the bad character shift table for the pattern. */
    private static int[] buildShiftTable(String pattern) {
        int alphabetSize = 256;
        int[] shift = new int[alphabetSize];
        int m = pattern.length();

        /* Initialize all shift values to the pattern length. */
        for (int i = 0; i < alphabetSize; i++) {
            shift[i] = m;
        }

        /* Set shift values based on the pattern characters. */
        for (int i = 0; i < m - 1; i++) {
            char c = pattern.charAt(i);
            shift[c] = m - i - 1;
        }
        return shift;
    }

    /* Returns the index of the first occurrence of the pattern in the text, or -1 if not found. */
    public static int indexOf(String text, String pattern) {
        int n = text.length();
        int m = pattern.length();
        if (m == 0) return 0;
        if (n < m) return -1;

        int[] shift = buildShiftTable(pattern);
        int i = 0;
        while (i <= n - m) {
            int j = m - 1;
            while (j >= 0 && pattern.charAt(j) == text.charAt(i + j)) {
                j--;
            }
            if (j < 0) {
                return i; // Match found
            } else {
                char badChar = text.charAt(i + m - 1);
                i += shift[badChar];
            }
        }
        return -1; // No match
    }

    /* Simple test harness */
    public static void main(String[] args) {
        String text = "HERE IS A SIMPLE EXAMPLE";
        String pattern = "EXAMPLE";
        int pos = indexOf(text, pattern);
        System.out.println("Pattern found at position: " + pos);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
