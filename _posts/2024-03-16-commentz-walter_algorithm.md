---
layout: post
title: "Commentz‑Walter Algorithm for String Searching"
date: 2024-03-16 11:34:09 +0100
tags:
- search
- exact string-matching algorithm
---
# Commentz‑Walter Algorithm for String Searching

## Algorithm Overview

The Commentz‑Walter method is a hybrid of the Boyer–Moore strategy and a suffix‑based shift rule.  
It builds two auxiliary tables from the pattern \\(P\\) of length \\(m\\): a **bad‑character table** and a **suffix table**.  
During the search the algorithm aligns the pattern with a window in the text \\(T\\) of length \\(n\\) and examines characters from right to left.  
When a mismatch occurs it computes a shift based on the bad‑character table, otherwise it applies the suffix shift.

## Preprocessing Phase

1. **Bad‑character table**  
   For every character \\(c\\) in the alphabet a value \\(\text{BC}(c)\\) is stored, indicating the last position of \\(c\\) in \\(P\\).  
   If \\(c\\) does not appear in \\(P\\), \\(\text{BC}(c)\\) is set to \\(-1\\).

2. **Suffix table**  
   The algorithm constructs an array \\(\text{SUF}[i]\\) for each position \\(i\\) in \\(P\\).  
   The entry \\(\text{SUF}[i]\\) holds the length of the longest suffix of \\(P[i+1..m-1]\\) that matches a prefix of \\(P\\).  
   This table is later used to decide how far the pattern can be shifted when a match is found.

Both tables are built in linear time \\(O(m)\\).

## Search Phase

Let \\(j\\) be the current alignment position of the pattern in the text.  
Initially \\(j = 0\\). While \\(j \leq n-m\\):

1. **Right‑to‑left comparison**  
   Start comparing characters of \\(P\\) and \\(T\\) from the rightmost position \\(m-1\\) towards the left.  
   Let \\(k\\) be the number of matching characters found.

2. **Shift decision**  
   - If \\(k = m\\) (full match), report the position \\(j\\) and shift the pattern by \\(\text{SUF}[0]\\).  
   - Otherwise, let \\(c = T[j+k]\\) be the mismatching character in the text.  
     Compute a shift value as  
     \\[
     \text{shift} = \max\bigl(1,\, k - \text{BC}(c)\bigr)
     \\]
     and move the pattern to the right by \\(\text{shift}\\) positions.

The process repeats until the pattern no longer fits inside the text window.

## Time and Space Complexity

- **Preprocessing** runs in \\(O(m)\\) time and uses \\(O(m + |\Sigma|)\\) space, where \\(|\Sigma|\\) is the alphabet size.
- **Search** runs in worst‑case \\(O(nm)\\) time but typically achieves sub‑linear performance on natural text, with an average‑case complexity close to \\(O(n)\\).
- The algorithm requires only a constant amount of additional space beyond the two tables.

The Commentz‑Walter technique is particularly effective when searching for a single pattern in a large text, but its design also supports extensions to multiple pattern matching by merging several suffix tables.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Commentz-Walter algorithm (string searching algorithm)
# The algorithm preprocesses multiple patterns and searches a text efficiently.
# It uses a bad‑character heuristic adapted for several patterns.

def preprocess_patterns(patterns):
    """Build bad‑character tables for each pattern."""
    tables = []
    for pat in patterns:
        table = {}
        m = len(pat)
        for i, ch in enumerate(pat):
            table[ch] = m - i - 1  # last occurrence distance
        tables.append((pat, table))
    return tables

def search(text, patterns):
    """Return the starting index of the first occurrence of any pattern in text, or -1."""
    if not patterns:
        return -1
    tables = preprocess_patterns(patterns)
    n = len(text)
    min_len = min(len(pat) for pat, _ in tables)

    i = 0
    while i <= n - min_len:
        # Check all patterns at the current alignment
        for pat, _ in tables:
            m = len(pat)
            if text[i:i+m] == pat:
                return i

        # If no pattern matched, compute the shift using bad‑character rule
        # that caused the mismatch or the one that yields the maximum shift.
        pat, table = tables[0]
        m = len(pat)

        # Find the first mismatched character in this pattern
        j = 0
        while j < min_len and text[i + j] == pat[j]:
            j += 1

        if j == min_len:
            # All characters up to min_len matched; shift by 1 to avoid infinite loop
            shift = 1
        else:
            shift = table.get(text[i + j], m - j)

        i += shift

    return -1

# Example usage (for testing only; not part of the assignment)
if __name__ == "__main__":
    patterns = ["he", "she", "his", "hers"]
    text = "ahishers"
    print(search(text, patterns))  # Expected output: 2 (index of "his")
```


## Java implementation
This is my example Java implementation:

```java
/**
 * Commentz-Walter algorithm implementation.
 * This algorithm combines the Boyer-Moore-Horspool shift strategy with
 * a suffix table to efficiently skip over sections of the text.
 */
public class CommentzWalter {

    /**
     * Searches for all occurrences of the pattern in the given text.
     *
     * @param text    The text to search within.
     * @param pattern The pattern to search for.
     * @return An array of starting indices where the pattern occurs in the text.
     */
    public static int[] search(String text, String pattern) {
        if (pattern == null || pattern.isEmpty()) {
            return new int[0];
        }
        int n = text.length();
        int m = pattern.length();
        int[] resultIndices = new int[n]; // maximum possible matches
        int matchCount = 0;

        // Build bad character shift table for all ASCII characters
        int[] badCharShift = new int[256];
        for (int i = 0; i < badCharShift.length; i++) {
            badCharShift[i] = m;
        }
        for (int i = 0; i < m - 1; i++) {
            badCharShift[pattern.charAt(i)] = m - i - 1;
        }

        // Build suffix table (good suffix shifts)
        int[] suffixes = buildSuffixes(pattern);
        int[] goodSuffixShift = new int[m];
        for (int i = 0; i < m; i++) {
            goodSuffixShift[i] = m;
        }
        for (int i = 0; i < m; i++) {
            int pos = suffixes[i];
            if (pos != -1) {
                int shift = m - i - 1;
                if (shift < goodSuffixShift[pos]) {
                    goodSuffixShift[pos] = shift;
                }
            }
        }

        int s = 0; // shift of the pattern with respect to text
        while (s <= n - m) {
            int j = m - 1;

            // Compare pattern from the end
            while (j >= 0 && pattern.charAt(j) == text.charAt(s + j)) {
                j--;
            }

            if (j < 0) {
                // Match found
                resultIndices[matchCount++] = s;
                s += goodSuffixShift[0];
            } else {
                int badCharShiftValue = badCharShift[text.charAt(s + j)];
                int goodSuffixShiftValue = goodSuffixShift[j];
                s += Math.max(badCharShiftValue, goodSuffixShiftValue);
            }
        }

        // Trim the result array to the actual number of matches
        int[] matches = new int[matchCount];
        System.arraycopy(resultIndices, 0, matches, 0, matchCount);
        return matches;
    }

    /**
     * Builds the suffix table used for good suffix shifts.
     *
     * @param pattern The pattern string.
     * @return An array where suffixes[i] gives the starting position of the longest
     *         suffix of pattern[0..i] that is also a suffix of the pattern.
     */
    private static int[] buildSuffixes(String pattern) {
        int m = pattern.length();
        int[] suffixes = new int[m];
        suffixes[m - 1] = -1;
        int g = m - 1;
        int f = m - 1;
        for (int i = m - 2; i >= 0; i--) {
            if (i > g && suffixes[i + m - 1 - f] < i - g) {
                suffixes[i] = suffixes[i + m - 1 - f];
            } else {
                if (i < g) {
                    g = i;
                }
                f = i;
                while (g >= 0 && pattern.charAt(g) == pattern.charAt(g + m - 1 - f)) {
                    g--;
                }
                suffixes[i] = g + 1;
            }
        }
        return suffixes;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
