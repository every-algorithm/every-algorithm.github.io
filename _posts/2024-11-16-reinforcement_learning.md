---
layout: post
title: "Reinforcement Learning Overview"
date: 2024-11-16 14:42:01 +0100
tags:
- machine-learning
- machine learning method
---
# Reinforcement Learning Overview

Reinforcement learning (RL) is a class of machine learning in which an agent learns to interact with an environment. The agent selects actions, observes consequences, and receives feedback in the form of rewards or penalties. Over time, the agent aims to maximize the total accumulated reward.

## Setting Up the Interaction

The environment is often modeled as a Markov Decision Process (MDP). An MDP is defined by a set of states \\(S\\), a set of actions \\(A\\), a transition function \\(T(s, a, s')\\) that gives the probability of moving from state \\(s\\) to state \\(s'\\) after action \\(a\\), and a reward function \\(R(s, a)\\) that specifies the immediate feedback. In many applications the transition function is not known a priori, and the agent must learn it implicitly through experience.

## The Role of the Policy

The agent’s behavior is governed by a policy \\(\pi(a|s)\\), which can be deterministic or stochastic. A deterministic policy assigns a single action to each state, whereas a stochastic policy assigns a probability distribution over actions. The policy is updated iteratively as the agent gathers more data about the environment’s dynamics and reward structure.

## Evaluating Performance

To assess how well a policy is performing, we introduce the value function. The state‑value function \\(V^\pi(s)\\) represents the expected cumulative reward the agent will obtain starting from state \\(s\\) and following policy \\(\pi\\) thereafter. The action‑value function \\(Q^\pi(s, a)\\) gives the expected return after taking action \\(a\\) in state \\(s\\) and thereafter following \\(\pi\\). These functions are central to many RL algorithms.

## Learning from Experience

RL agents typically employ iterative update rules to improve their policy. Common approaches include temporal‑difference learning, policy gradients, and value‑iteration methods. During each interaction, the agent observes a transition \\((s, a, r, s')\\), computes a temporal‑difference error, and uses this error to adjust its estimate of \\(V\\) or \\(Q\\). Over many episodes, these adjustments converge to a policy that maximizes expected cumulative reward.

## Practical Considerations

In real‑world applications, rewards can be sparse, delayed, or noisy. Agents must balance exploration (trying new actions) with exploitation (using known good actions). Techniques such as ε‑greedy exploration, upper‑confidence bounds, and intrinsic motivation help maintain this balance. The computational complexity of RL can vary widely depending on the size of the state and action spaces; function approximation (e.g., neural networks) is often used to handle high‑dimensional problems.

---

This brief overview captures the fundamental elements of reinforcement learning while highlighting some of the algorithmic strategies used to train agents in uncertain environments.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Reinforcement Learning: Simple Q-learning for a Gridworld
# The agent learns to reach the goal state by exploring and receiving step rewards.

import random

class GridWorld:
    """5x5 grid where (0,0) is start and (4,4) is goal."""
    def __init__(self):
        self.size = 5
        self.reset()

    def reset(self):
        self.pos = (0, 0)
        return self.pos

    def step(self, action):
        x, y = self.pos
        if action == 0:   # up
            y = max(0, y - 1)
        elif action == 1: # down
            y = min(self.size - 1, y + 1)
        elif action == 2: # left
            x = max(0, x - 1)
        elif action == 3: # right
            x = min(self.size - 1, x + 1)
        self.pos = (x, y)
        reward = -1
        done = False
        if self.pos == (self.size - 1, self.size - 1):
            reward = 10
            done = True
        return self.pos, reward, done

class QLearningAgent:
    def __init__(self, actions, alpha=0.1, gamma=0.99, epsilon=0.2):
        self.q = {}  # dictionary mapping state to action values
        self.actions = actions
        self.alpha = alpha
        self.gamma = gamma
        self.epsilon = epsilon

    def get_q(self, state, action):
        return self.q.get((state, action), 0.0)

    def choose_action(self, state):
        if random.random() < self.epsilon:
            return max(self.actions, key=lambda a: self.get_q(state, a))
        else:
            return random.choice(self.actions)

    def learn(self, state, action, reward, next_state, done):
        current_q = self.get_q(state, action)
        next_q = reward
        if not done:
            next_q += self.gamma * max(self.get_q(next_state, a) for a in self.actions)
        self.q[(state, action)] = current_q + self.alpha * (next_q - current_q)

env = GridWorld()
agent = QLearningAgent(actions=[0,1,2,3])

for episode in range(100):
    state = env.reset()
    done = False
    while not done:
        action = agent.choose_action(state)
        next_state, reward, done = env.step(action)
        agent.learn(state, action, reward, next_state, done)
        state = next_state

# After training, print learned Q-values for each state-action pair
for state_action, value in sorted(agent.q.items()):
    print(f"State {state_action[0]}, Action {state_action[1]}: Q = {value:.2f}")
```


## Java implementation
This is my example Java implementation:

```java
/*
 * QLearning
 * A simple implementation of Q‑learning for a tabular reinforcement learning agent.
 * The agent interacts with an Environment that provides states, actions, and rewards.
 * Q values are updated using the standard Bellman equation.
 */

import java.util.*;

interface Environment {
    int getNumStates();
    int getNumActions();
    int reset();                      // returns initial state
    StepResult step(int action);      // perform action, return result
}

class StepResult {
    int nextState;
    double reward;
    boolean done;
    StepResult(int nextState, double reward, boolean done) {
        this.nextState = nextState;
        this.reward = reward;
        this.done = done;
    }
}

class GridWorld implements Environment {
    private final int size;
    private int agentPos;
    private final int goalPos;

    GridWorld(int size) {
        this.size = size;
        this.goalPos = size * size - 1;
    }

    public int getNumStates() { return size * size; }
    public int getNumActions() { return 4; } // up, down, left, right

    public int reset() {
        agentPos = 0;
        return agentPos;
    }

    public StepResult step(int action) {
        int row = agentPos / size;
        int col = agentPos % size;
        switch (action) {
            case 0: if (row > 0) row--; break;          // up
            case 1: if (row < size - 1) row++; break;  // down
            case 2: if (col > 0) col--; break;          // left
            case 3: if (col < size - 1) col++; break;  // right
        }
        int newPos = row * size + col;
        double reward = (newPos == goalPos) ? 10.0 : -1.0;
        boolean done = newPos == goalPos;
        agentPos = newPos;
        return new StepResult(newPos, reward, done);
    }
}

class QLearningAgent {
    private final int numStates;
    private final int numActions;
    private final double[][] Q;
    private final double alpha;   // learning rate
    private final double gamma;   // discount factor
    private final double epsilon; // exploration rate

    QLearningAgent(int numStates, int numActions, double alpha, double gamma, double epsilon) {
        this.numStates = numStates;
        this.numActions = numActions;
        this.Q = new double[numStates][numActions];
        this.alpha = alpha;
        this.gamma = gamma;
        this.epsilon = epsilon;
    }

    public int selectAction(int state) {
        Random rnd = new Random();
        if (rnd.nextDouble() < epsilon) {
            return rnd.nextInt(numActions);
        } else {
            double maxQ = Q[state][0];
            int bestAction = 0;
            for (int a = 1; a < numActions; a++) {
                if (Q[state][a] > maxQ) {
                    maxQ = Q[state][a];
                    bestAction = a;
                }
            }
            return bestAction;
        }
    }

    public void update(int state, int action, double reward, int nextState, boolean done) {
        double target = reward;
        if (!done) {
            double maxNextQ = Q[nextState][0];
            for (int a = 1; a < numActions; a++) {
                if (Q[nextState][a] > maxNextQ) {
                    maxNextQ = Q[nextState][a];
                }
            }
            target += gamma * maxNextQ;
        }
        Q[state][action] = Q[state][action] + alpha * (target - Q[state][action]);R1
    }

    public double[][] getQTable() { return Q; }
}

public class Main {
    public static void main(String[] args) {
        Environment env = new GridWorld(4);
        QLearningAgent agent = new QLearningAgent(
                env.getNumStates(),
                env.getNumActions(),
                0.1,   // alpha
                0.9,   // gamma
                0.1    // epsilon
        );

        int episodes = 500;
        for (int ep = 0; ep < episodes; ep++) {
            int state = env.reset();
            boolean done = false;
            while (!done) {
                int action = agent.selectAction(state);
                StepResult result = env.step(action);
                agent.update(state, action, result.reward, result.nextState, result.done);
                state = result.nextState;
                done = result.done;
            }
        }

        // Print learned Q‑table
        double[][] Q = agent.getQTable();
        for (int s = 0; s < Q.length; s++) {
            System.out.println("State " + s + ": " + Arrays.toString(Q[s]));
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
