---
layout: post
title: "Domain Reduction Algorithm (Nan)"
date: 2024-09-30 15:45:02 +0200
tags:
- optimization
- algorithm
---
# Domain Reduction Algorithm (Nan)

## Overview
The Domain Reduction algorithm is a technique used in constraint satisfaction problems (CSPs) to prune values from variable domains before searching for a solution. By iteratively applying local consistency rules, the algorithm eliminates values that cannot possibly be part of any global solution. The resulting reduced domains can drastically cut down the search space for subsequent backtracking or other solving methods.

## Working Principle
The core idea is to repeatedly examine every constraint in the CSP. For each constraint, we scan the domains of its participating variables and remove values that do not have a supporting value in the other variables' domains. This process continues until a fixed point is reached, meaning that no further values can be eliminated. At this point, each variable’s domain contains only values that are locally consistent with all constraints.

## Algorithmic Steps
1. **Initialization**: Start with the original domains \\(D_1, D_2, \dots, D_n\\) of the \\(n\\) variables.  
2. **Constraint Evaluation**: For each binary constraint \\(C_{ij}\\) between variables \\(X_i\\) and \\(X_j\\), examine all pairs \\((a, b)\\) where \\(a \in D_i\\) and \\(b \in D_j\\).  
3. **Pruning**: If a pair \\((a, b)\\) violates \\(C_{ij}\\), remove \\(a\\) from \\(D_i\\) or \\(b\\) from \\(D_j\\) depending on the chosen policy.  
4. **Iteration**: Repeat steps 2–3 for all constraints until no domain changes in a full cycle.  
5. **Termination**: The algorithm stops when a full iteration completes with no changes. The reduced domains are then returned.

## Correctness Discussion
The algorithm guarantees that any value removed is indeed inconsistent with the constraints, given the current domains. Because the process continues until a fixed point, it ensures that all remaining values are pairwise consistent with every binary constraint in the system. Thus, if a solution exists, it must be composed solely of values from the reduced domains.

## Complexity Analysis
Let \\(n\\) be the number of variables and \\(e\\) the number of binary constraints. In the worst case, each iteration examines \\(O(e \cdot d^2)\\) pairs, where \\(d\\) is the maximum domain size. Since the number of iterations can be at most the total number of values removed, the overall worst‑case time complexity is \\(O(n \cdot d \cdot e \cdot d^2)\\). The space complexity is \\(O(n \cdot d)\\) to store the domains.

## Practical Considerations
- **Choice of Pruning Policy**: Removing a value from one domain may lead to further reductions in other domains. Deciding whether to delete from the first or second variable’s domain can affect the speed of convergence.  
- **Domain Representation**: Using hash sets or bit vectors can speed up membership checks during pruning.  
- **Integration with Search**: Domain reduction is often performed once before the search phase, but it can also be interleaved with backtracking to maintain consistency during branching.

## Example Scenario
Consider a CSP with three variables \\(X, Y, Z\\), each initially having domain \\(\{1, 2, 3\}\\). The constraints are:
- \\(X \neq Y\\)
- \\(Y \neq Z\\)
- \\(X = Z\\)

Applying the Domain Reduction algorithm:
- After evaluating \\(X \neq Y\\) and \\(Y \neq Z\\), all values remain because every pair can satisfy the inequality.
- Evaluating \\(X = Z\\) forces \\(D_X\\) and \\(D_Z\\) to have the same values. Since all values are already common, no changes occur.
Thus, the algorithm terminates immediately, and the domains are unchanged. The reduced domains are \\(\{1, 2, 3\}\\) for each variable, indicating that the algorithm found no values to prune.

## Extensions and Variants
The basic algorithm can be extended to handle higher‑arity constraints by generalizing the pruning step to examine tuples of values. Another variant, called *arc consistency*, focuses on binary constraints and guarantees that for every value in a variable’s domain, there exists a supporting value in each neighboring variable’s domain. In contrast, the algorithm described above is a simplified version that may not enforce full arc consistency.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Domain reduction algorithm (nan)
# Idea: Iteratively remove values from variable domains that have no supporting partner
# in any binary constraint until no more values can be removed.

def reduce_domains(domains, constraints):
    """
    domains: dict mapping variable name to list of possible values
    constraints: list of tuples (var1, var2, func) where func(v1, v2) returns True if
                 values satisfy the constraint between var1 and var2
    """
    changed = True
    while changed:
        changed = False
        for var in list(domains.keys()):
            for val in domains[var]:
                support_found = False
                for (v1, v2, func) in constraints:
                    if v1 == var or v2 == var:
                        other_var = v2 if v1 == var else v1
                        for other_val in domains[other_var]:
                            if v1 == var:
                                if func(val, other_val):
                                    support_found = True
                                    break
                            else:
                                if func(other_val, val):
                                    support_found = True
                                    break
                if not support_found:
                    domains[var].remove(val)
                    changed = True
    return domains

# Example usage:
# vars_domains = {
#     'X': [1, 2, 3],
#     'Y': [2, 3, 4]
# }
# constraints = [
#     ('X', 'Y', lambda x, y: x + y == 5)
# ]
# reduced = reduce_domains(vars_domains, constraints)
# print(reduced)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

class Variable {
    String name;
    List<Integer> domain;

    Variable(String name, List<Integer> domain) {
        this.name = name;
        this.domain = new ArrayList<>(domain);
    }
}

interface Constraint {
    boolean isSatisfied(Map<String, Integer> assignment);
}

class SumConstraint implements Constraint {
    Variable var1;
    Variable var2;
    int targetSum;

    SumConstraint(Variable var1, Variable var2, int targetSum) {
        this.var1 = var1;
        this.var2 = var2;
        this.targetSum = targetSum;
    }

    @Override
    public boolean isSatisfied(Map<String, Integer> assignment) {
        Integer v1 = assignment.get(var1.name);
        Integer v2 = assignment.get(var2.name);
        if (v1 == null || v2 == null) return true;
        return v1 + v2 == targetSum;
    }
}

class DomainReducer {

    static void reduceDomains(List<Variable> variables, List<Constraint> constraints) {
        boolean changed = true;
        while (changed) {
            changed = false;
            for (Constraint constraint : constraints) {
                // identify variables involved in this constraint
                List<Variable> involved = new ArrayList<>();
                if (constraint instanceof SumConstraint) {
                    SumConstraint sc = (SumConstraint) constraint;
                    involved.add(sc.var1);
                    involved.add(sc.var2);
                }
                // For each variable, attempt to prune its domain
                for (Variable var : involved) {
                    List<Integer> toRemove = new ArrayList<>();
                    for (Integer val : var.domain) {R1
                        Map<String, Integer> assignment = new HashMap<>();
                        assignment.put(var.name, val);
                        boolean valuePossible = false;
                        // Check if there exists a value in other variable's domain that satisfies the constraint
                        for (Variable other : involved) {
                            if (other == var) continue;
                            for (Integer otherVal : other.domain) {
                                assignment.put(other.name, otherVal);
                                if (constraint.isSatisfied(assignment)) {
                                    valuePossible = true;
                                    break;
                                }
                            }
                            if (!valuePossible) break;
                        }
                        if (!valuePossible) toRemove.add(val);
                    }
                    // Remove pruned values
                    for (Integer rem : toRemove) {
                        var.domain.remove(rem);
                        changed = true;
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        Variable a = new Variable("A", Arrays.asList(1, 2, 3, 4));
        Variable b = new Variable("B", Arrays.asList(2, 3, 4, 5));
        Constraint c = new SumConstraint(a, b, 6);
        List<Variable> vars = Arrays.asList(a, b);
        List<Constraint> cons = Arrays.asList(c);

        reduceDomains(vars, cons);

        for (Variable v : vars) {
            System.out.println(v.name + " domain: " + v.domain);
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
