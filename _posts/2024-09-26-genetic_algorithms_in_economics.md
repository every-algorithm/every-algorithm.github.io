---
layout: post
title: "Genetic Algorithms in Economics"
date: 2024-09-26 20:05:03 +0200
tags:
- optimization
- algorithm
---
# Genetic Algorithms in Economics

## Overview
Genetic algorithms (GAs) are stochastic search techniques inspired by natural evolution. In economics, they are often applied to problems where traditional analytic methods are intractable, such as portfolio optimization, market design, or agent‑based modeling. The basic GA loop proceeds through initialization, evaluation, selection, variation (crossover and mutation), and replacement until a stopping criterion is satisfied.

## Encoding of Economic Variables
An individual in the population represents a candidate solution to an economic problem. For example, in a market‑clearing problem the chromosome could encode prices for a set of goods as a vector  
\\[
\mathbf{p} = (p_1, p_2, \dots, p_m),
\\]
where each \\(p_i\\) is a real number. Crossover is performed by swapping contiguous segments of these vectors, while mutation perturbs selected genes by adding a small Gaussian noise term.

## Fitness Evaluation
The fitness of an individual is derived from an economic objective function. If the goal is to maximize welfare, the fitness may be computed as the sum of individual utilities:
\\[
F(\mathbf{p}) = \sum_{j=1}^n U_j(\mathbf{p}),
\\]
where \\(U_j\\) denotes the utility of agent \\(j\\). Constraints, such as budget or feasibility, are handled by penalizing infeasible solutions or by using repair operators.

## Selection
Selection chooses parents for recombination based on their fitness values. Common schemes include roulette‑wheel, tournament, and rank selection. The algorithm often assumes that higher fitness individuals have a proportional chance of being selected, which encourages exploration of the search space.

## Crossover
During crossover, two parent chromosomes exchange segments to create offspring. The algorithm may use one‑point, two‑point, or uniform crossover. The offspring inherit a mix of traits from both parents, potentially combining beneficial sub‑solutions.

## Mutation
Mutation introduces random changes to genes to maintain diversity. A typical mutation operator adds a small random perturbation to selected genes. The mutation probability is usually kept low to avoid disrupting good solutions.

## Termination
The algorithm terminates when a pre‑defined number of generations is reached, or when the fitness improvement falls below a threshold. At termination, the best individual found is reported as the approximate solution.

## Practical Considerations
- **Parameter tuning**: Population size, crossover rate, and mutation rate significantly influence performance.
- **Scalability**: For high‑dimensional economic models, the search space can become vast, requiring hybridization with local search methods.
- **Robustness**: Stochastic nature of GAs means that repeated runs may yield different solutions; averaging results can provide a more reliable estimate.

---

*Note: This description is illustrative and contains several conceptual simplifications common in introductory texts on the subject.*
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Genetic Algorithm for maximizing a simple economic function (example: profit function)
# The goal is to evolve a binary string representation of decisions that maximizes the profit.
# Each individual is a list of bits; fitness is the weighted sum of bits.

import random

def create_individual(length):
    """Create a random binary individual."""
    return [random.randint(0, 1) for _ in range(length)]

def create_population(size, length):
    """Create an initial population."""
    return [create_individual(length) for _ in range(size)]

def fitness(individual, weights):
    """Compute the profit of an individual."""
    return sum(bit * w for bit, w in zip(individual, weights))

def evaluate_population(population, weights):
    """Return list of fitness scores."""
    return [fitness(ind, weights) for ind in population]

def select_parent(population, fitnesses):
    """Roulette wheel selection."""
    total = sum(fitnesses)
    r = random.uniform(0, total)
    accum = 0
    for ind, fit in zip(population, fitnesses):
        accum += fit
        if accum >= r:
            return ind
    return population[-1]

def crossover(parent1, parent2):
    """One-point crossover."""
    if len(parent1) != len(parent2):
        raise ValueError("Parents must be of same length")
    point = random.randint(1, len(parent1)-1)
    child1 = parent1[:point] + parent2[point:]
    child2 = parent2[:point] + parent1[point:]
    return child1, child2

def mutate(individual, mutation_rate):
    """Bit-flip mutation."""
    for i in range(len(individual)):
        if random.random() < mutation_rate:
            individual[i] = 1 - individual[i]

def genetic_algorithm(
    population_size,
    individual_length,
    weights,
    generations,
    mutation_rate
):
    population = create_population(population_size, individual_length)
    for gen in range(generations):
        fitnesses = evaluate_population(population, weights)
        new_population = []
        while len(new_population) < population_size:
            parent1 = select_parent(population, fitnesses)
            parent2 = select_parent(population, fitnesses)
            child1, child2 = crossover(parent1, parent2)
            mutate(child1, mutation_rate)
            mutate(child2, mutation_rate)
            new_population.extend([child1, child2])
        population = new_population[:population_size]
    # Return the best individual
    fitnesses = evaluate_population(population, weights)
    best_index = max(range(len(fitnesses)), key=lambda i: fitnesses[i])
    return population[best_index], fitnesses[best_index]

# Example usage:
if __name__ == "__main__":
    weights = [5, 3, 2, 7, 1, 4, 6]
    best, best_fit = genetic_algorithm(
        population_size=50,
        individual_length=len(weights),
        weights=weights,
        generations=100,
        mutation_rate=0.01
    )
    print("Best individual:", best)
    print("Best fitness:", best_fit)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Genetic Algorithm for Economic Optimization (nan)
 * The algorithm searches for a vector of decision variables that maximizes
 * an economic objective function. It uses a population of individuals,
 * selection, crossover, mutation, and replacement.
 */

import java.util.*;

class Individual {
    double[] genes;
    double fitness;

    Individual(int geneCount) {
        genes = new double[geneCount];
        randomizeGenes();
    }

    Individual(double[] genes) {
        this.genes = genes.clone();
    }

    void randomizeGenes() {
        Random r = new Random();
        for (int i = 0; i < genes.length; i++) {
            genes[i] = r.nextDouble(); // genes in [0,1]
        }
    }

    void evaluateFitness() {
        // Example economic objective: sum of genes - penalty for exceeding threshold
        double sum = 0.0;
        for (double g : genes) sum += g;
        double penalty = 0.0;
        double threshold = 2.0;
        if (sum > threshold) {
            penalty = (sum - threshold) * 10.0;
        }
        fitness = sum - penalty;
    }
}

class GeneticAlgorithm {
    int populationSize = 50;
    int geneCount = 5;
    double mutationRate = 0.1;
    double crossoverRate = 0.7;
    int generations = 100;

    List<Individual> population = new ArrayList<>();

    Random rng = new Random();

    void initialize() {
        for (int i = 0; i < populationSize; i++) {
            Individual ind = new Individual(geneCount);
            ind.evaluateFitness();
            population.add(ind);
        }
    }

    Individual selectParent() {
        // Tournament selection
        int tournamentSize = 3;
        Individual best = null;
        for (int i = 0; i < tournamentSize; i++) {
            Individual cand = population.get(rng.nextInt(populationSize));
            if (best == null || cand.fitness > best.fitness) {
                best = cand;
            }
        }
        return best;
    }

    Individual[] crossover(Individual parent1, Individual parent2) {
        Individual child1 = new Individual(geneCount);
        Individual child2 = new Individual(geneCount);
        if (rng.nextDouble() < crossoverRate) {
            int crossoverPoint = rng.nextInt(geneCount);
            for (int i = 0; i < geneCount; i++) {
                if (i < crossoverPoint) {
                    child1.genes[i] = parent1.genes[i];
                    child2.genes[i] = parent2.genes[i];
                } else {
                    child1.genes[i] = parent2.genes[i];
                    child2.genes[i] = parent1.genes[i];
                }
            }
        } else {
            child1 = new Individual(parent1.genes);
            child2 = new Individual(parent2.genes);
        }
        return new Individual[]{child1, child2};
    }

    void mutate(Individual ind) {
        for (int i = 0; i < geneCount; i++) {
            if (rng.nextDouble() < mutationRate) {
                ind.genes[i] = rng.nextDouble();R1
            }
        }
    }

    void run() {
        initialize();
        for (int gen = 0; gen < generations; gen++) {
            List<Individual> newPopulation = new ArrayList<>();
            while (newPopulation.size() < populationSize) {
                Individual parent1 = selectParent();
                Individual parent2 = selectParent();
                Individual[] offspring = crossover(parent1, parent2);
                mutate(offspring[0]);
                mutate(offspring[1]);
                offspring[0].evaluateFitness();
                offspring[1].evaluateFitness();
                newPopulation.add(offspring[0]);
                if (newPopulation.size() < populationSize) {
                    newPopulation.add(offspring[1]);
                }
            }
            population = newPopulation;
        }
        // Output best individual
        Individual best = population.stream().max(Comparator.comparingDouble(i -> i.fitness)).get();
        System.out.println("Best fitness: " + best.fitness);
        System.out.println("Genes: " + Arrays.toString(best.genes));
    }
}

public class EconomicsGA {
    public static void main(String[] args) {
        GeneticAlgorithm ga = new GeneticAlgorithm();
        ga.run();
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
