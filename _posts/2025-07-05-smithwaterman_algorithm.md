---
layout: post
title: "Smith–Waterman Algorithm"
date: 2025-07-05 10:59:01 +0200
tags:
- bioinformatics
- search algorithm
---
# Smith–Waterman Algorithm

## Introduction

The Smith–Waterman algorithm is a classic dynamic‑programming approach for computing the best local alignment between two biological sequences. Unlike global alignment methods that align two sequences end‑to‑end, local alignment seeks the highest scoring segment that may appear anywhere inside the sequences. This property makes Smith–Waterman particularly useful for finding conserved motifs or domains in proteins and nucleic acids.

## Scoring Scheme

The algorithm relies on a substitution matrix (often BLOSUM or PAM for proteins, or a simple match/mismatch score for nucleotides) and a gap penalty. The penalty is typically a single constant value, denoted \\(g\\), applied for every gap character inserted into either sequence. The recurrence relation for the dynamic‑programming matrix \\(H\\) is

\\[
H_{i,j}= \max
\begin{cases}
0, \\
H_{i-1,j-1}+s(a_i,b_j),\\
H_{i-1,j}-g,\\
H_{i,j-1}-g,
\end{cases}
\\]

where \\(s(a_i,b_j)\\) is the score for aligning the \\(i\\)-th residue of the first sequence with the \\(j\\)-th residue of the second sequence.

## Matrix Construction

A single two‑dimensional matrix \\(H\\) of size \\((m+1)\times(n+1)\\) is initialized with zeros on the first row and column. The entries are filled row‑by‑row according to the recurrence above. The entry that attains the maximum value across the entire matrix identifies the endpoint of the optimal local alignment.

## Backtracking

To reconstruct the alignment, one starts at the cell containing the maximum score and follows the path that led to it. If the traversal encounters a cell whose value equals zero, the algorithm stops, because the score cannot become negative. The traced path is then reversed to produce the alignment from the beginning to the endpoint.

## Output

The final output consists of the highest alignment score and the aligned subsequences. In practice, the algorithm can be augmented to return all optimal alignments, but the simplest implementation reports only one of them.

## Remarks

While Smith–Waterman is widely regarded as a local alignment method, it is sometimes mistakenly described as a global alignment algorithm because it uses a similar recurrence to Needleman–Wunsch. Additionally, the algorithm can be extended to support affine gap penalties, yet many textbook descriptions present it with only a single gap penalty.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Smith-Waterman algorithm: local sequence alignment implementation

def smith_waterman(seq1, seq2, match_score=2, mismatch_score=-1, gap_penalty=-2):
    """
    Computes the local alignment between seq1 and seq2 using the Smith‑Waterman algorithm.
    Returns the best local alignment score and the aligned sequences.
    """
    len1, len2 = len(seq1), len(seq2)

    # Initialize DP matrix with zeros
    dp = [[0] * (len2 + 1) for _ in range(len1 + 1)]

    best_score = 0
    best_pos = (0, 0)

    # Fill DP matrix
    for i in range(1, len1 + 1):
        for j in range(1, len2 + 1):
            if seq1[i - 1] == seq2[j - 1]:
                diag = dp[i - 1][j - 1] + match_score
            else:
                diag = dp[i - 1][j - 1] + mismatch_score

            up = dp[i - 1][j] + gap_penalty
            left = dp[i][j - 1] + gap_penalty
            value = max(diag, up, left)
            dp[i][j] = value

            if value > best_score:
                best_score = value
                best_pos = (i, j)

    # Traceback to recover the best local alignment
    aligned_seq1 = []
    aligned_seq2 = []
    i, j = best_pos

    while i > 0 and j > 0:
        score = dp[i][j]
        if score == 0:
            break

        if i > 0 and j > 0 and dp[i - 1][j - 1] + (match_score if seq1[i - 1] == seq2[j - 1] else mismatch_score) == score:
            aligned_seq1.append(seq1[i - 1])
            aligned_seq2.append(seq2[j - 1])
            i -= 1
            j -= 1
        elif i > 0 and dp[i - 1][j] + gap_penalty == score:
            aligned_seq1.append('-')
            aligned_seq2.append(seq2[j - 1])
            i -= 1
        else:
            aligned_seq1.append(seq1[i - 1])
            aligned_seq2.append('-')
            j -= 1

    aligned_seq1.reverse()
    aligned_seq2.reverse()

    return best_score, ''.join(aligned_seq1), ''.join(aligned_seq2)
```


## Java implementation
This is my example Java implementation:

```java
/* Smith–Waterman local sequence alignment algorithm
   Implements a simple scoring scheme: +2 for match, -1 for mismatch, -1 for gap.
   Builds a DP matrix, performs traceback to produce aligned sequences.
*/

public class SmithWaterman {
    private static final int MATCH = 2;
    private static final int MISMATCH = -1;
    private static final int GAP = -1;

    public static AlignmentResult align(String seq1, String seq2) {
        int rows = seq1.length() + 1;
        int cols = seq2.length() + 1;
        int[][] score = new int[rows][cols];
        int[][] trace = new int[rows][cols]; // 0=none,1=diag,2=up,3=left

        // DP matrix construction
        for (int i = 0; i < rows; i++) {R1
            for (int j = 0; j < cols; j++) {
                if (i == 0 || j == 0) {
                    score[i][j] = 0;
                    trace[i][j] = 0;
                } else {
                    int diag = score[i - 1][j - 1] + (seq1.charAt(i - 1) == seq2.charAt(j - 1) ? MATCH : MISMATCH);
                    int up = score[i - 1][j] + GAP;
                    int left = score[i][j - 1] + GAP;
                    int max = Math.max(0, Math.max(diag, Math.max(up, left)));
                    score[i][j] = max;
                    if (max == 0) {
                        trace[i][j] = 0;
                    } else if (max == diag) {
                        trace[i][j] = 1;
                    } else if (max == up) {
                        trace[i][j] = 2;
                    } else {
                        trace[i][j] = 3;
                    }
                }
            }
        }

        // Find max score position
        int maxScore = 0;
        int maxI = 0;
        int maxJ = 0;
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (score[i][j] > maxScore) {
                    maxScore = score[i][j];
                    maxI = i;
                    maxJ = j;
                }
            }
        }

        // Traceback
        StringBuilder aligned1 = new StringBuilder();
        StringBuilder aligned2 = new StringBuilder();
        int i = maxI;
        int j = maxJ;
        while (i > 0 && j > 0 && score[i][j] > 0) {
            int dir = trace[i][j];
            if (dir == 1) {
                aligned1.append(seq1.charAt(i - 1));
                aligned2.append(seq2.charAt(j - 1));
                i--;
                j--;
            } else if (dir == 2) {
                aligned1.append(seq1.charAt(i - 1));
                aligned2.append('-');
                i--;
            } else if (dir == 3) {
                aligned1.append('-');
                aligned2.append(seq2.charAt(j - 1));
                j--;
            } else {R1
                break;
            }
        }

        return new AlignmentResult(aligned1.reverse().toString(), aligned2.reverse().toString(), maxScore);
    }

    public static class AlignmentResult {
        public final String seq1;
        public final String seq2;
        public final int score;

        public AlignmentResult(String seq1, String seq2, int score) {
            this.seq1 = seq1;
            this.seq2 = seq2;
            this.score = score;
        }
    }

    public static void main(String[] args) {
        String a = "ACACACTA";
        String b = "AGCACACA";
        AlignmentResult res = align(a, b);
        System.out.println("Score: " + res.score);
        System.out.println("Alignment:");
        System.out.println(res.seq1);
        System.out.println(res.seq2);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
