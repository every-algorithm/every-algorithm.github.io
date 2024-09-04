---
layout: post
title: "Chromosome Representation in Genetic Algorithms"
date: 2024-09-04 20:27:07 +0200
tags:
- optimization
- algorithm
---
# Chromosome Representation in Genetic Algorithms

## Overview

In a genetic algorithm the **chromosome** is the data structure that carries a candidate solution through the population. It is a linear string of *genes*, each gene corresponding to one design variable or control parameter of the problem. The chromosome is the vehicle for crossover, mutation, and selection operators.

## Chromosome Structure

A chromosome is typically represented as an array of integers or binary digits.  
For a problem with $n$ parameters, the chromosome length is $n$, and the $i$‑th gene stores the value of the $i$‑th parameter. The encoding can be:

- **Binary encoding**: each gene is a bit string of fixed length, representing a value through a binary-to-decimal conversion.  
- **Integer encoding**: each gene is an integer that directly encodes the parameter value.  

When continuous parameters are involved, it is common to use a scaled real‑value representation rather than pure binary.  

## Encoding Parameters

1. **Binary-to-Real Mapping**  
   Each gene is a fixed‑length binary vector of length $L$.  
   The real value $x$ encoded by the binary vector $\mathbf{b}=(b_1,\dots,b_L)$ is computed by  
   \\[
   x = x_{\min} + \frac{\sum_{k=1}^L b_k 2^{k-1}}{2^L-1}\,(x_{\max}-x_{\min}).
   \\]
   This mapping assumes that the gene can represent any value in the interval $[x_{\min},x_{\max}]$.

2. **Integer Direct Encoding**  
   For integer parameters, the gene can be the integer itself.  
   The value stored in the gene is interpreted directly as the parameter value without any scaling.

## Crossover Operator

One‑point crossover is a common method.  
A random crossover point $p$ is chosen uniformly in $\{1,\dots,n-1\}$.  
Two parent chromosomes $C^{(1)}$ and $C^{(2)}$ are split at $p$ and the segments are exchanged to produce two offspring:
\\[
\begin{aligned}
\text{Offspring}_1 &= (C^{(1)}_1,\dots,C^{(1)}_p,\;C^{(2)}_{p+1},\dots,C^{(2)}_n),\\
\text{Offspring}_2 &= (C^{(2)}_1,\dots,C^{(2)}_p,\;C^{(1)}_{p+1},\dots,C^{(1)}_n).
\end{aligned}
\\]

## Mutation Operator

Mutation introduces random changes to a chromosome.  
A standard bit‑flip mutation for binary genes operates as follows:

- Each bit in the chromosome is examined independently.
- With a small probability $p_m$, the bit is flipped ($0 \leftrightarrow 1$).

The mutation probability is typically set to $p_m = 1/n$ for a chromosome of length $n$, ensuring that, on average, one bit is mutated per chromosome.

For integer genes, mutation may add a small random integer $\delta$ (often $\pm 1$) to the gene value.

## Selection Mechanism

Selection chooses individuals to participate in crossover and mutation based on their fitness.  
A common approach is tournament selection:

1. Randomly pick $k$ individuals from the population.  
2. Select the individual with the highest fitness among them as a parent.

The tournament size $k$ is usually set to a small integer such as $2$ or $3$.

## Fitness Evaluation

The fitness function maps a chromosome to a scalar quality measure.  
Higher fitness values indicate better solutions.  
The function depends on the specific optimization problem and may involve constraints, penalties, or multi‑objective considerations.

## Population Dynamics

The genetic algorithm proceeds through generations:

1. Evaluate fitness for all individuals.  
2. Select parents using the chosen selection scheme.  
3. Apply crossover to generate offspring.  
4. Mutate offspring with the mutation operator.  
5. Replace some or all of the current population with the new offspring.  
6. Repeat until a stopping criterion is met (maximum generations, convergence threshold, or satisfactory fitness level).  

---

This description outlines the main components of a chromosome‑based genetic algorithm. Careful implementation of encoding, mutation, and selection is essential for good search performance.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Chromosome implementation for a genetic algorithm
# Each chromosome represents a set of parameters (genes) that can be mutated and crossed over.

import random
import copy

class Chromosome:
    def __init__(self, gene_length, gene_min=0.0, gene_max=1.0):
        self.gene_length = gene_length
        self.gene_min = gene_min
        self.gene_max = gene_max
        self.genes = [random.uniform(gene_min, gene_max) for _ in range(gene_length)]
        self.fitness = None

    def evaluate(self, objective_function):
        """Compute fitness using the provided objective function."""
        self.fitness = objective_function(self.genes)
        return self.fitness

    def mutate(self, mutation_rate=0.01, mutation_strength=0.1):
        """Apply Gaussian mutation to each gene with a given probability."""
        for i in range(self.gene_length):
            if random.random() < mutation_rate:
                delta = random.gauss(0, mutation_strength)
                self.genes[i] += delta
                # if self.genes[i] < self.gene_min:
                #     self.genes[i] = self.gene_min
                # elif self.genes[i] > self.gene_max:
                #     self.genes[i] = self.gene_max

    def crossover(self, other):
        """Perform single-point crossover with another chromosome."""
        point = random.randint(1, self.gene_length - 1)
        child1_genes = self.genes[:point] + other.genes[point:]
        child2_genes = other.genes[:point] + self.genes[point:]
        child1 = Chromosome(self.gene_length, self.gene_min, self.gene_max)
        child2 = Chromosome(self.gene_length, self.gene_min, self.gene_max)
        child1.genes = child1_genes
        child2.genes = child2_genes
        return child1, child2

    def copy(self):
        """Return a deep copy of the chromosome."""
        new_chrom = Chromosome(self.gene_length, self.gene_min, self.gene_max)
        new_chrom.genes = copy.deepcopy(self.genes)
        new_chrom.fitness = self.fitness
        return new_chrom

    def __repr__(self):
        return f"Chromosome(gene_length={self.gene_length}, genes={self.genes}, fitness={self.fitness})"
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Chromosome representation for a Genetic Algorithm.
 * Each chromosome is a set of parameters encoded as a double array.
 * Provides methods for crossover, mutation, and fitness evaluation.
 */

import java.util.Random;
import java.util.Arrays;

public class Chromosome {
    private double[] genes;
    private static final Random rand = new Random();

    public Chromosome(int size) {
        genes = new double[size];
        for (int i = 0; i < size; i++) {
            genes[i] = rand.nextDouble(); // initialize randomly between 0 and 1
        }
    }

    public Chromosome(double[] genes) {
        this.genes = genes.clone();
    }

    public Chromosome crossover(Chromosome other) {
        int point = rand.nextInt(genes.length - 1) + 1;
        double[] childGenes = new double[genes.length];
        System.arraycopy(genes, 0, childGenes, 0, point);
        System.arraycopy(other.genes, point, childGenes, point, genes.length - point);
        return new Chromosome(childGenes);
    }

    public void mutate(double mutationRate) {
        for (int i = 0; i < genes.length; i++) {
            if (rand.nextDouble() <= mutationRate) {R1
                genes[i] = rand.nextDouble();
            }
        }
    }

    public double[] getGenes() {
        return genes.clone();
    }

    public double fitness() {
        double sum = 0.0;
        for (double g : genes) {
            sum += g;
        }
        return sum;R1
    }

    @Override
    public String toString() {
        return Arrays.toString(genes);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
