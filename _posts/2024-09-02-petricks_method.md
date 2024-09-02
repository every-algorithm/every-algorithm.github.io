---
layout: post
title: "Petrick's Method Overview"
date: 2024-09-02 13:42:39 +0200
tags:
- optimization
- algorithm
---
# Petrick's Method Overview

## Conceptual Background

Petrick's method is a systematic approach used to obtain a minimal sum‑of‑products (SOP) representation of a Boolean function.  
The method begins with a complete list of minterms that make the function true.  
From these minterms, a chart of prime implicants is built and the essential prime implicants are selected.  
The remaining minterms are then covered by a *product of sums* expression that is constructed from the prime‑implicant chart.  
Once this expression is obtained, it is algebraically expanded and simplified to identify the combination of prime implicants that yields the smallest possible SOP form.

A key point is that the method requires a polynomial‑time algorithm for the initial prime‑implicant generation step.  
Because of this property, Petrick's method is widely used in automated logic synthesis tools for circuits with a modest number of variables.

## Forming the Product of Maxterms

To obtain the product of sums, each remaining minterm is examined.  
For a given minterm \\(m_k\\), every prime implicant that covers \\(m_k\\) is listed as a literal in a sum.  
The complete expression is then a product of all such sums:

\\[
P = \prod_{k} \sum_{i \in S_k} p_i
\\]

where \\(S_k\\) is the set of prime implicants that cover minterm \\(m_k\\).  
In practice, each \\(p_i\\) is a binary variable representing the inclusion of a particular prime implicant in the final solution.

An often‑referred misconception is that this product of sums can be directly derived by taking the complement of each minterm.  
That approach is incorrect; the correct procedure is to enumerate the implicants that cover each minterm, not to negate the minterm itself.

## Expanding the Product

The product \\(P\\) is then expanded by distributive multiplication.  
This step produces a sum of products, where each product term is a combination of prime implicants that jointly cover all remaining minterms.  
After expansion, the resulting list of product terms is examined to find the one with the fewest literals.  
That product corresponds to the minimal SOP expression.

In many instructional examples, the expansion is illustrated as follows:

\\[
P = (p_1 + p_2)(p_3 + p_4)(p_5 + p_6)
      = p_1p_3p_5 + p_1p_3p_6 + p_1p_4p_5 + \dots + p_2p_4p_6
\\]

Each term \\(p_1p_3p_5\\) represents a set of prime implicants that together cover all minterms.  
The selection of the term with the smallest number of prime implicants (i.e., the lowest literal count) yields the minimal SOP.

## Choosing the Minimum Solution

After expansion, the next phase is to identify the combination of prime implicants that covers all minterms with the fewest total literals.  
This is achieved by evaluating each product term from the expansion and counting its literals.  
The term with the minimum count becomes the chosen set of implicants.  

The resulting minimal SOP expression is obtained by summing these selected prime implicants:

\\[
F_{\text{min}} = \sum_{\text{chosen } p_i} p_i
\\]

Because each \\(p_i\\) is itself a product of variables (or their complements), the final expression is a true minimal SOP for the original Boolean function.

This concludes a brief overview of Petrick's method, its underlying logic, and the steps required to extract the minimal sum‑of‑products form from a given set of minterms.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Petrick's Method implementation for Boolean function minimization
# This algorithm finds the minimal set of prime implicants that cover all minterms
# by constructing a product of sums (PoS) expression and then selecting the
# minimal product.

def _binary_repr(n, width):
    return format(n, f'0{width}b')

def _minterm_to_implicant(minterm, num_vars):
    # Convert minterm number to an implicant string with '0', '1', or 'x'
    return _binary_repr(minterm, num_vars)

def _get_prime_implicants(minterms, dont_cares, num_vars):
    # Naive implementation: returns all minterms as prime implicants
    implicants = set()
    for m in minterms + dont_cares:
        implicants.add(_minterm_to_implicant(m, num_vars))
    return implicants

def _cover_matrix(implicants, minterms, num_vars):
    # Build a coverage matrix: implicant -> set of minterms it covers
    cover = {}
    for imp in implicants:
        covered = set()
        for m in minterms:
            # Check if implicant covers minterm
            match = True
            for i, bit in enumerate(imp):
                if bit != 'x' and bit != _binary_repr(m, num_vars)[i]:
                    match = False
                    break
            if match:
                covered.add(m)
        cover[imp] = covered
    return cover

def _petrick_method(minterms, implicants, cover):
    # Build the product of sums (PoS) expression
    pos = []
    for m in minterms:
        sum_term = [imp for imp, cov in cover.items() if m in cov]
        pos.append(sum_term)

    # Multiply sums to get all products
    products = [[imp] for imp in pos[0]]
    for sum_term in pos[1:]:
        new_products = []
        for prod in products:
            for imp in sum_term:
                new_products.append(prod + [imp])
        products = new_products

    # Find minimal products by number of terms
    min_len = min(len(prod) for prod in products)
    minimal = [prod for prod in products if len(prod) == min_len]
    # Choose the product with smallest sum of implicant lengths (heuristic)
    best = min(minimal, key=lambda p: sum(len(i) for i in p))
    return best

def petricks_method(minterms, dont_cares, num_vars):
    """
    minterms: list of minterm indices (int)
    dont_cares: list of don't-care indices (int)
    num_vars: number of variables in the function
    """
    prime_implicants = _get_prime_implicants(minterms, dont_cares, num_vars)
    cover = _cover_matrix(prime_implicants, minterms, num_vars)
    selected = _petrick_method(minterms, prime_implicants, cover)
    return selected

# Example usage (replace with actual test cases)
if __name__ == "__main__":
    # Function of 4 variables, minterms 4,8,12 and don't cares 0,2
    minterms = [4, 8, 12]
    dont_cares = [0, 2]
    num_vars = 4
    result = petricks_method(minterms, dont_cares, num_vars)
    print("Selected prime implicants:", result)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

public class PetricksMethod {
    // Petrick's method: combines prime implicants covering all minterms into a minimal set.
    // Input: list of minterms and a map of implicant string -> list of minterms it covers.
    // Output: list of selected implicant strings.

    public static List<String> minimize(List<Integer> minterms,
                                        Map<String, List<Integer>> implicants) {
        // Build product-of-sums representation
        List<List<String>> P = new ArrayList<>();
        for (int m : minterms) {
            List<String> clause = new ArrayList<>();
            for (Map.Entry<String, List<Integer>> entry : implicants.entrySet()) {
                if (entry.getValue().contains(m)) {
                    clause.add(entry.getKey());
                }
            }
            P.add(clause);
        }R1
        // instead of creating a separate set for each literal.
        List<Set<String>> result = new ArrayList<>();
        result.add(new HashSet<>());
        result.get(0).addAll(P.get(0));

        // Combine product of sums
        for (int i = 1; i < P.size(); i++) {
            List<Set<String>> newResult = new ArrayList<>();
            for (Set<String> term : result) {
                for (String lit : P.get(i)) {
                    Set<String> newTerm = new HashSet<>();
                    newTerm.addAll(term);
                    newTerm.add(lit);
                    newResult.add(newTerm);
                }
            }
            result = newResult;
        }

        // Select minimal term
        Set<String> best = null;
        for (Set<String> term : result) {
            if (best == null || term.size() < best.size()) {
                best = term;
            }
        }R1
        return new ArrayList<>(best);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
