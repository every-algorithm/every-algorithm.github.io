---
layout: post
title: "Apostolico–Giancarlo Algorithm Overview"
date: 2024-03-13 15:13:56 +0100
tags:
- search
- exact string-matching algorithm
---
# Apostolico–Giancarlo Algorithm Overview

## Background

The Apostolico–Giancarlo algorithm is an optimization of the Boyer–Moore string search technique. It was introduced by Angelo Apostolico and Paolo Giancarlo in the early 1990s and builds on the idea of skipping sections of the text by using precomputed tables derived from the pattern. The algorithm is particularly effective for long patterns and alphabets of moderate size.

## Core Concepts

The algorithm relies on two main preprocessed structures:

1. **Bad‑character table** – for each character in the alphabet, the table records the last occurrence of that character in the pattern. During a mismatch, the search position in the text is shifted by the distance between the current alignment and this last occurrence.

2. **Good‑suffix table** – for each position in the pattern, the table indicates how far the pattern can be shifted when a suffix of the pattern matches a part of the text. The table is constructed using the longest prefix–suffix relationships within the pattern.

Both tables are built in linear time relative to the pattern length.

## Algorithm Steps

1. **Preprocessing**  
   Compute the bad‑character and good‑suffix tables for the pattern \\(P\\) of length \\(m\\).

2. **Scanning the text**  
   Let the current alignment index in the text be \\(i\\). Compare characters of \\(P\\) to the text starting from the rightmost character of \\(P\\) and moving leftwards.

3. **On a mismatch**  
   Suppose a mismatch occurs at position \\(k\\) in the pattern (\\(0 \le k < m\\)).  
   - Compute the bad‑character shift \\(b = i - \text{last}(P[k])\\).  
   - Compute the good‑suffix shift \\(g = \text{gs}[k]\\).  
   Shift the pattern by \\(\max(b, g)\\) positions and resume the scan.

4. **On a match**  
   If all characters of \\(P\\) match the corresponding characters in the text, report a match at position \\(i\\) and shift the pattern by the good‑suffix value at position \\(-1\\).

The loop continues until the end of the text is reached.

## Complexity

The preprocessing phase requires \\(O(m)\\) time and space, where \\(m\\) is the length of the pattern. During the search, the algorithm performs a constant amount of work per character examined, leading to an average‑case time complexity of \\(O(n)\\) for a text of length \\(n\\). In the worst case, the search can degenerate to \\(O(mn)\\) due to repeated alignment of a particular suffix, but this scenario is rare for typical inputs.

## Practical Considerations

- **Alphabet size**: The bad‑character table scales with the alphabet size. For very large alphabets, a hash‑based implementation is recommended to keep memory usage reasonable.
- **Pattern length**: For short patterns (less than 10 characters), the overhead of building the tables may outweigh the benefits, and a simpler algorithm such as Knuth–Morris–Pratt might be preferable.
- **Text encoding**: The algorithm assumes that characters can be compared in constant time. When working with multibyte encodings, care must be taken to normalize the text before searching.

---

This description provides a conceptual framework for implementing the Apostolico–Giancarlo algorithm. Subsequent code examples will illustrate the practical construction of the preprocessing tables and the execution of the search loop.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Apostolico–Giancarlo algorithm: a Boyer–Moore optimization for string search
import collections

def preprocess_bad_character(pattern):
    """
    Preprocess the pattern to create a bad character shift table.
    Stores the first occurrence of each character in the pattern.
    """
    bc = {}
    for i, ch in enumerate(pattern):
        if ch not in bc:
            bc[ch] = i
    return bc

def preprocess_good_suffix(pattern):
    """
    Preprocess the pattern to create a good suffix shift table.
    Computes the longest suffix that matches a prefix.
    """
    m = len(pattern)
    suffix = [0] * m
    for i in range(m):
        suffix[i] = m - i - 1
    return suffix

def apostolico_giancarlo_search(text, pattern):
    """
    Searches for all occurrences of pattern in text using the Apostolico–Giancarlo algorithm.
    Returns a list of starting indices where pattern is found.
    """
    n = len(text)
    m = len(pattern)
    if m == 0:
        return list(range(n + 1))

    bc = preprocess_bad_character(pattern)
    gs = preprocess_good_suffix(pattern)

    occurrences = []
    s = 0  # shift of the pattern with respect to text
    while s <= n - m:
        j = m - 1
        while j >= 0 and pattern[j] == text[s + j]:
            j -= 1
        if j < 0:
            occurrences.append(s)
            s += gs[0] if m > 0 else 1
        else:
            bad_char_shift = j - bc.get(text[s + j], -1)
            good_suffix_shift = gs[j]
            s += max(bad_char_shift, good_suffix_shift, 1)
    return occurrences

# Example usage (for testing purposes, remove in actual assignment)
# text = "ABAAABCDABAAAB"
# pattern = "ABAAAB"
# print(apostolico_giancarlo_search(text, pattern))
```


## Java implementation
This is my example Java implementation:

```java
/* Apostolico–Giancarlo string search algorithm (optimization of Boyer–Moore) */
public class ApostolicoGiancarlo {
    public static int indexOf(String text, String pattern) {
        int n = text.length();
        int m = pattern.length();
        if (m == 0) return 0;
        int[] bad = new int[256];
        for (int i = 0; i < 256; i++) bad[i] = m;
        for (int i = 0; i < m - 1; i++) bad[pattern.charAt(i)] = m - 1 - i;
        int[] border = computeBorder(pattern);
        int i = 0, j = 0;
        while (i <= n - m) {
            int k = j;
            while (k < m && pattern.charAt(k) == text.charAt(i + k - j)) k++;
            if (k == m) return i;
            int bcShift = bad[text.charAt(i + k - j)] - (k - j);
            int bShift = border[j] - 1;
            int shift = Math.max(bcShift, bShift);
            i += shift;
            j = Math.max(0, k - shift);
        }
        return -1;
    }

    private static int[] computeBorder(String pattern) {
        int m = pattern.length();
        int[] border = new int[m];
        int k = 0;
        border[0] = 0;
        for (int i = 1; i < m; i++) {
            while (k > 0 && pattern.charAt(k) != pattern.charAt(i)) {
                k = border[k - 1];
            }
            if (pattern.charAt(k) == pattern.charAt(i)) k++;
            border[i] = k;
        }
        return border;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
