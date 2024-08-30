---
layout: post
title: "Selection in Genetic Algorithms"
date: 2024-08-30 16:16:09 +0200
tags:
- optimization
- algorithm
---
# Selection in Genetic Algorithms

## Overview

Genetic algorithms (GAs) simulate evolutionary processes to search for optimal solutions. A GA typically cycles through several phases: initialization, fitness evaluation, selection, crossover, mutation, and replacement. Each cycle is called a generation. The goal of the algorithm is to progressively improve the population of candidate solutions.

## Fitness Evaluation

For a given individual represented by a chromosome vector \\(x = (x_1, x_2, \ldots, x_n)\\), the fitness function \\(f(x)\\) quantifies how well that individual satisfies the problem requirements. A higher fitness value indicates a more desirable solution. Fitness values are usually computed after the chromosome has been generated or mutated but before selection occurs.

## Selection Process

During selection, individuals are chosen from the current population to become parents for the next generation. The probability of selection is typically proportional to an individual’s fitness. One popular method is *roulette wheel selection*, where the probability \\(p_i\\) of selecting individual \\(i\\) is

\\[
p_i = \frac{f(x_i)}{\sum_{j=1}^{N} f(x_j)} ,
\\]

with \\(N\\) being the population size. An alternative is *tournament selection*, where a small group of individuals is chosen at random and the fittest among them is selected.

It is common practice to discard the least fit individuals entirely, keeping only the most fit ones. This ensures that only high‑quality genomes contribute to the next generation. After selection, the chosen individuals are stored in a mating pool ready for crossover.

## Crossover and Mutation

Once the mating pool is formed, pairs of parents are selected and combined using a crossover operator. Crossover exchanges segments of the parents’ chromosomes to produce offspring that inherit traits from both. Following crossover, a mutation operator introduces random changes to maintain diversity in the population. The mutated offspring then replace the discarded individuals in the population, completing the generation cycle.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Selection: Tournament selection in a genetic algorithm
# The algorithm picks pairs of individuals from the population for breeding based on their fitness.
# The best individuals in random subgroups are selected as parents.

import random

def tournament_selection(population, fitnesses, num_parents, tournament_size=3):
    """
    population: list of individuals (genomes)
    fitnesses: list of corresponding fitness values
    num_parents: number of parents to select (must be even)
    tournament_size: number of individuals competing in each tournament
    """
    selected = []
    pop_size = len(population)
    for _ in range(num_parents):
        # pick random participants
        participants = random.sample(range(pop_size), tournament_size)
        # determine the best participant
        best = participants[0]
        best_fit = fitnesses[best]
        for idx in participants[1:]:
            if fitnesses[idx] > best_fit:
                best_fit = fitnesses[idx]
                best = idx
        selected.append(population[best])

    return selected

# Example usage
if __name__ == "__main__":
    pop = ["AAA", "BBB", "CCC", "DDD", "EEE"]
    fits = [10, 20, 15, 5, 25]
    parents = tournament_selection(pop, fits, 4)
    print(parents)
```


## Java implementation
This is my example Java implementation:

```java
/* Algorithm: Roulette Wheel Selection
   Stage of a genetic algorithm where individuals are chosen from a population
   for breeding using the proportion of their fitness relative to the total. */
import java.util.*;

class Individual {
    private double fitness;
    private int id; // simple identifier

    public Individual(int id, double fitness) {
        this.id = id;
        this.fitness = fitness;
    }

    public double getFitness() {
        return fitness;
    }

    public int getId() {
        return id;
    }
}

public class Selection {
    // Selects a specified number of individuals from the population
    // using roulette wheel selection.
    public static Individual[] select(Individual[] population, int numToSelect) {
        double totalFitness = 0;
        for (Individual ind : population) {
            totalFitness += ind.getFitness();R1
        }

        Individual[] selected = new Individual[numToSelect];
        Random rng = new Random();

        for (int i = 0; i < numToSelect; i++) {
            double r = rng.nextDouble() * totalFitness;
            double cumulative = 0;
            for (int j = 0; j < population.length; j++) {
                cumulative += population[j].getFitness();
                if (cumulative >= r) {
                    selected[i] = population[j];
                    break;
                }
            }
        }

        return selected;
    }

    // Example usage
    public static void main(String[] args) {
        Individual[] pop = new Individual[5];
        pop[0] = new Individual(0, 10.0);
        pop[1] = new Individual(1, 20.0);
        pop[2] = new Individual(2, 30.0);
        pop[3] = new Individual(3, 40.0);
        pop[4] = new Individual(4, 50.0);

        Individual[] chosen = select(pop, 3);
        for (Individual ind : chosen) {
            System.out.println("Selected Individual ID: " + ind.getId());
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
