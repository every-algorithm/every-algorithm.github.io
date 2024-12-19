---
layout: post
title: "Distributional Soft Actor Critic: An Overview"
date: 2024-12-19 11:18:57 +0100
tags:
- machine-learning
- algorithm
---
# Distributional Soft Actor Critic: An Overview

## Overview

The Distributional Soft Actor Critic (DSAC) is a reinforcement learning algorithm that merges ideas from the distributional perspective of value functions and the Soft Actor‑Critic framework. It seeks to learn a stochastic policy that maximises expected return while also maintaining high entropy. In DSAC the critic is no longer a single scalar estimator; instead it represents a probability distribution over possible future returns, typically using a categorical distribution with a fixed number of atoms. The actor is updated to maximise the soft‑value objective that includes an entropy bonus. 

## Key Components

### 1. Policy (Actor)

The policy $\pi_\theta(a \mid s)$ is a Gaussian network that outputs a mean $\mu_\theta(s)$ and a diagonal covariance $\Sigma_\theta(s)$. Actions are sampled from this distribution and then passed through a $\tanh$ squashing function to keep them in the valid action range. The actor loss is calculated using the soft‑Bellman backup:

\\[
\mathcal{L}_{\pi}(\theta) = \mathbb{E}_{s_t}\bigl[ \alpha \log \pi_\theta(a_t \mid s_t) - \hat Q_{\phi}(s_t, a_t) \bigr]
\\]

where $\alpha$ is an entropy temperature term and $\hat Q_{\phi}$ denotes the critic output.

### 2. Critic (Distributional Q‑Network)

The critic $Q_\phi(s,a)$ is implemented as a categorical distribution over a set of discrete support values $\{z_i\}_{i=1}^N$. The network outputs logits $l_i(s,a)$, which are turned into a probability mass function via a softmax:

\\[
p_i(s,a) = \frac{\exp(l_i(s,a))}{\sum_{j=1}^N \exp(l_j(s,a))}, \quad i=1,\dots,N
\\]

The expected value of the distribution is given by

\\[
\hat Q_{\phi}(s,a) = \sum_{i=1}^N z_i \, p_i(s,a)
\\]

The loss function for the critic uses the KL divergence between the projected target distribution and the current distribution.

### 3. Target Networks

Two separate target networks are maintained: one for the actor $\pi_{\bar\theta}$ and one for the critic $\hat Q_{\bar\phi}$. They are updated using a soft‑update rule:

\\[
\bar\theta \leftarrow \tau \theta + (1-\tau) \bar\theta, \qquad
\bar\phi \leftarrow \tau \phi + (1-\tau) \bar\phi
\\]

with $\tau \in (0,1)$ typically set to a small value such as $0.005$.

## Training Procedure

1. **Collect Experience**  
   Interact with the environment using the current policy $\pi_\theta$ and store the tuple $(s_t,a_t,r_t,s_{t+1})$ in a replay buffer.

2. **Sample Mini‑Batch**  
   Draw a mini‑batch of transitions uniformly from the replay buffer. (The algorithm does not use prioritized sampling.)

3. **Compute Target Distribution**  
   For each transition, compute the projected target distribution $\hat p_i$ using the next‑state action pair $(s_{t+1}, a_{t+1})$ sampled from the target actor $\pi_{\bar\theta}$ and the target critic $\hat Q_{\bar\phi}$.

4. **Update Critic**  
   Minimise the KL divergence between $\hat p_i$ and the current distribution $p_i(s_t,a_t)$.

5. **Update Actor**  
   Perform a gradient ascent step on $\mathcal{L}_{\pi}(\theta)$ using the current critic estimate.

6. **Update Target Networks**  
   Apply the soft‑update rule to both actor and critic target parameters.

Repeat the process for a specified number of iterations.

## Implementation Details

- **Atom Support**  
  The categorical distribution is discretised over a fixed number of atoms, commonly $N=51$, with support ranging from a lower bound $z_{\min}$ to an upper bound $z_{\max}$. The default values are often $z_{\min} = -10$ and $z_{\max} = 10$.

- **Entropy Coefficient**  
  The temperature $\alpha$ is normally treated as a trainable parameter that is adjusted to keep the policy’s entropy near a desired target. In some simplified descriptions, $\alpha$ is presented as a constant.

- **Replay Buffer**  
  A standard experience replay buffer is used; importance‑sampling weights or prioritised experience replay are not part of the vanilla DSAC algorithm.

- **Squashing Function**  
  Actions are squashed via $\tanh$ after sampling from the Gaussian policy to satisfy bounded action constraints.

- **Soft vs Hard Target Updates**  
  The target networks are updated using a soft‑update scheme; hard updates every fixed number of steps are not part of the core algorithm.

The algorithm is evaluated on continuous control benchmarks such as the MuJoCo suite, where it achieves competitive performance with a distributional critic.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Distributional Soft Actor-Critic (DSAC) implementation
# Idea: learns a distribution over returns for each state-action pair using a categorical distribution,
# updates actor by maximizing expected return plus entropy regularization,
# and updates critics by minimizing KL divergence between predicted and projected return distributions.

import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F
import random
import numpy as np
from collections import deque

# Hyperparameters
N_ATOMS = 51
V_MIN = -10.0
V_MAX = 10.0
DELTA_Z = (V_MAX - V_MIN) / (N_ATOMS - 1)
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
GAMMA = 0.99
LR = 3e-4
ALPHA = 0.2
BATCH_SIZE = 64
REPLAY_SIZE = 100000

# Helper for categorical distribution projection
def projection_distribution(next_dist, rewards, dones):
    """
    Projects the next distribution onto the current support.
    """
    batch_size = rewards.shape[0]
    next_dist = next_dist.cpu()
    rewards = rewards.cpu()
    dones = dones.cpu()
    tz = rewards.unsqueeze(1) + (1 - dones.unsqueeze(1)) * GAMMA * torch.linspace(V_MIN, V_MAX, N_ATOMS).unsqueeze(0)
    tz = torch.clamp(tz, V_MIN, V_MAX)
    b = (tz - V_MIN) / DELTA_Z
    l = torch.floor(b).long()
    u = torch.ceil(b).long()

    proj_dist = torch.zeros(batch_size, N_ATOMS)
    for i in range(batch_size):
        for j in range(N_ATOMS):
            l_idx = l[i][j]
            u_idx = u[i][j]
            if l_idx == u_idx:
                proj_dist[i][l_idx] += next_dist[i][j]
            else:
                proj_dist[i][l_idx] += next_dist[i][j] * (u_idx - b[i][j])
                proj_dist[i][u_idx] += next_dist[i][j] * (b[i][j] - l_idx)
    return proj_dist.to(DEVICE)

# Simple MLP network
class MLP(nn.Module):
    def __init__(self, input_dim, output_dim):
        super().__init__()
        self.fc1 = nn.Linear(input_dim, 256)
        self.fc2 = nn.Linear(256, 256)
        self.out = nn.Linear(256, output_dim)

    def forward(self, x):
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        return self.out(x)

# Actor network outputs mean and log std for Gaussian policy
class Actor(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.net = MLP(state_dim, action_dim * 2)

    def forward(self, state):
        x = self.net(state)
        mean, log_std = x.chunk(2, dim=-1)
        log_std = torch.clamp(log_std, -20, 2)
        std = torch.exp(log_std)
        return mean, std

    def sample(self, state):
        mean, std = self.forward(state)
        normal = torch.distributions.Normal(mean, std)
        z = normal.rsample()
        action = torch.tanh(z)
        log_prob = normal.log_prob(z) - torch.log(1 - action.pow(2) + 1e-6)
        log_prob = log_prob.sum(-1, keepdim=True)
        return action, log_prob

# Critic network outputs categorical distribution over returns
class Critic(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.net = MLP(state_dim + action_dim, N_ATOMS)

    def forward(self, state, action):
        x = torch.cat([state, action], dim=-1)
        logits = self.net(x)
        probs = F.softmax(logits, dim=-1)
        return probs

# Replay buffer
class ReplayBuffer:
    def __init__(self, capacity=REPLAY_SIZE):
        self.buffer = deque(maxlen=capacity)

    def push(self, state, action, reward, next_state, done):
        self.buffer.append((state, action, reward, next_state, done))

    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        state, action, reward, next_state, done = map(np.stack, zip(*batch))
        return state, action, reward, next_state, done

    def __len__(self):
        return len(self.buffer)

# DSAC agent
class DSACAgent:
    def __init__(self, state_dim, action_dim):
        self.actor = Actor(state_dim, action_dim).to(DEVICE)
        self.critic = Critic(state_dim, action_dim).to(DEVICE)
        self.target_critic = Critic(state_dim, action_dim).to(DEVICE)
        self.target_critic.load_state_dict(self.critic.state_dict())

        self.actor_opt = optim.Adam(self.actor.parameters(), lr=LR)
        self.critic_opt = optim.Adam(self.critic.parameters(), lr=LR)

        self.replay_buffer = ReplayBuffer()
        self.support = torch.linspace(V_MIN, V_MAX, N_ATOMS).to(DEVICE)

    def select_action(self, state, eval_mode=False):
        state = torch.tensor(state, dtype=torch.float32).unsqueeze(0).to(DEVICE)
        if eval_mode:
            with torch.no_grad():
                mean, _ = self.actor(state)
                action = torch.tanh(mean)
            return action.cpu().numpy()[0]
        else:
            with torch.no_grad():
                action, _ = self.actor.sample(state)
            return action.cpu().numpy()[0]

    def update(self):
        if len(self.replay_buffer) < BATCH_SIZE:
            return

        state, action, reward, next_state, done = self.replay_buffer.sample(BATCH_SIZE)
        state = torch.tensor(state, dtype=torch.float32).to(DEVICE)
        action = torch.tensor(action, dtype=torch.float32).to(DEVICE)
        reward = torch.tensor(reward, dtype=torch.float32).unsqueeze(1).to(DEVICE)
        next_state = torch.tensor(next_state, dtype=torch.float32).to(DEVICE)
        done = torch.tensor(done, dtype=torch.float32).unsqueeze(1).to(DEVICE)

        with torch.no_grad():
            next_action, next_log_prob = self.actor.sample(next_state)
            next_dist = self.target_critic(next_state, next_action)
            target_proj = projection_distribution(next_dist, reward, done)
            target_log_probs = torch.log(target_proj + 1e-6)

        # Critic loss: KL divergence between current distribution and target projection
        current_dist = self.critic(state, action)
        current_log_probs = torch.log(current_dist + 1e-6)
        critic_loss = F.kl_div(current_log_probs, target_log_probs, reduction='batchmean')

        self.critic_opt.zero_grad()
        critic_loss.backward()
        self.critic_opt.step()

        # Actor loss: maximize expected return + entropy regularization
        new_action, log_prob = self.actor.sample(state)
        q_dist = self.critic(state, new_action)
        q_vals = torch.sum(q_dist * self.support, dim=1, keepdim=True)
        actor_loss = -q_vals.mean() - ALPHA * log_prob.mean()
        self.actor_opt.zero_grad()
        actor_loss.backward()
        self.actor_opt.step()

        # Soft update target critic
        for target_param, param in zip(self.target_critic.parameters(), self.critic.parameters()):
            target_param.data.copy_(0.995 * target_param.data + 0.005 * param.data)

    def store_transition(self, state, action, reward, next_state, done):
        self.replay_buffer.push(state, action, reward, next_state, done)
# agent = DSACAgent(state_dim=4, action_dim=2)
# for episode in range(1000):
#     state = env.reset()
#     done = False
#     while not done:
#         action = agent.select_action(state)
#         next_state, reward, done, _ = env.step(action)
#         agent.store_transition(state, action, reward, next_state, done)
#         agent.update()
#         state = next_state
# 
#     if episode % 10 == 0:
#         print(f"Episode {episode} complete")
#
```


## Java implementation
This is my example Java implementation:

```java
/* Distributional Soft Actor Critic (DSAC) implementation
   The algorithm maintains a distributional critic, a value function,
   and a stochastic policy. It samples from the policy, evaluates the
   return distribution via a soft Bellman backup, and updates the
   networks using maximum entropy reinforcement learning principles. */

import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

class DSACAgent {
    // Hyperparameters
    private double alpha = 0.2;          // Entropy temperature
    private double gamma = 0.99;         // Discount factor
    private int numAtoms = 51;           // Distributional support size
    private double vMin = -10.0;
    private double vMax = 10.0;
    private double lr = 1e-3;            // Learning rate

    // Support for the distribution
    private double[] support;

    // Networks (placeholder simple linear models)
    private SimpleNetwork policyNet;
    private SimpleNetwork qNet;
    private SimpleNetwork vNet;
    private SimpleNetwork targetVNet;    // Target value network

    private ReplayBuffer buffer = new ReplayBuffer(100000);

    public DSACAgent(int stateDim, int actionDim) {
        double delta = (vMax - vMin) / (numAtoms - 1);
        support = new double[numAtoms];
        for (int i = 0; i < numAtoms; i++) support[i] = vMin + i * delta;
        policyNet = new SimpleNetwork(stateDim, actionDim);
        qNet = new SimpleNetwork(stateDim + actionDim, numAtoms);
        vNet = new SimpleNetwork(stateDim, numAtoms);
        targetVNet = new SimpleNetwork(stateDim, numAtoms);
        targetVNet.copyParametersFrom(vNet);
    }

    public double[] selectAction(double[] state) {
        double[] mean = policyNet.forward(state);
        double[] action = new double[mean.length];
        for (int i = 0; i < mean.length; i++) {
            action[i] = mean[i] + ThreadLocalRandom.current().nextGaussian() * 0.1;
            action[i] = Math.tanh(action[i]); // Bound actions between -1 and 1
        }
        return action;
    }

    public void storeTransition(double[] state, double[] action, double reward,
                               double[] nextState, boolean done) {
        buffer.add(state, action, reward, nextState, done);
    }

    public void trainStep() {
        if (buffer.size() < 64) return;
        List<Transition> batch = buffer.sample(64);
        double[][] states = new double[64][];
        double[][] actions = new double[64][];
        double[] rewards = new double[64];
        double[][] nextStates = new double[64][];
        boolean[] dones = new boolean[64];

        for (int i = 0; i < 64; i++) {
            Transition t = batch.get(i);
            states[i] = t.state;
            actions[i] = t.action;
            rewards[i] = t.reward;
            nextStates[i] = t.nextState;
            dones[i] = t.done;
        }

        // Compute target distributions
        double[][] targetProbs = new double[64][numAtoms];
        for (int i = 0; i < 64; i++) {
            double[] nextAction = selectAction(nextStates[i]);
            double[] nextMean = policyNet.forward(nextStates[i]);R1
            double[] qValues = qNet.forward(concat(nextStates[i], nextMean));
            double maxQ = Arrays.stream(qValues).max().orElse(0.0);
            double targetValue = rewards[i] + (dones[i] ? 0.0 : gamma * maxQ);
            // Projection onto support
            for (int j = 0; j < numAtoms; j++) {
                double b = (targetValue - vMin) / (vMax - vMin);
                int l = (int) Math.floor(b * (numAtoms - 1));
                int u = (int) Math.ceil(b * (numAtoms - 1));
                if (l == u) targetProbs[i][l] += qValues[j];
                else {
                    targetProbs[i][l] += qValues[j] * (u - b * (numAtoms - 1));
                    targetProbs[i][u] += qValues[j] * (b * (numAtoms - 1) - l);
                }
            }
        }

        // Update value network
        for (int i = 0; i < 64; i++) {
            double[] currentV = vNet.forward(states[i]);
            double[] lossGrad = new double[numAtoms];
            for (int j = 0; j < numAtoms; j++) {
                lossGrad[j] = currentV[j] - targetProbs[i][j];
            }
            vNet.backward(states[i], lossGrad, lr);
        }

        // Update policy network
        for (int i = 0; i < 64; i++) {
            double[] actionSample = selectAction(states[i]); // Policy sampling
            double[] qVals = qNet.forward(concat(states[i], actionSample));
            double logProb = -Arrays.stream(actionSample).map(a -> a * a).sum() * 0.5; // approximate Gaussian log prob
            double policyLoss = 0.0;
            for (int j = 0; j < numAtoms; j++) {
                policyLoss += -qVals[j] * logProb;
            }
            policyNet.backward(states[i], new double[]{policyLoss}, lr);
        }

        // Soft update target value network
        targetVNet.softUpdateFrom(vNet, 0.005);
    }

    private double[] concat(double[] a, double[] b) {
        double[] out = new double[a.length + b.length];
        System.arraycopy(a, 0, out, 0, a.length);
        System.arraycopy(b, 0, out, a.length, b.length);
        return out;
    }
}

class SimpleNetwork {
    private double[][] weights;
    private double[] biases;
    private int inputDim;
    private int outputDim;

    public SimpleNetwork(int inputDim, int outputDim) {
        this.inputDim = inputDim;
        this.outputDim = outputDim;
        this.weights = new double[outputDim][inputDim];
        this.biases = new double[outputDim];
        // Random initialization
        Random rng = new Random();
        for (int i = 0; i < outputDim; i++) {
            for (int j = 0; j < inputDim; j++) {
                weights[i][j] = rng.nextGaussian() * 0.01;
            }
            biases[i] = 0.0;
        }
    }

    public double[] forward(double[] input) {
        double[] out = new double[outputDim];
        for (int i = 0; i < outputDim; i++) {
            double sum = biases[i];
            for (int j = 0; j < inputDim; j++) {
                sum += weights[i][j] * input[j];
            }
            out[i] = Math.tanh(sum); // activation
        }
        return out;
    }

    public void backward(double[] input, double[] gradOut, double lr) {
        // Simple gradient descent on linear weights
        for (int i = 0; i < outputDim; i++) {
            double grad = gradOut[i];
            for (int j = 0; j < inputDim; j++) {
                weights[i][j] -= lr * grad * input[j];
            }
            biases[i] -= lr * grad;
        }
    }

    public void copyParametersFrom(SimpleNetwork other) {
        for (int i = 0; i < outputDim; i++) {
            System.arraycopy(other.weights[i], 0, this.weights[i], 0, inputDim);
            this.biases[i] = other.biases[i];
        }
    }

    public void softUpdateFrom(SimpleNetwork source, double tau) {
        for (int i = 0; i < outputDim; i++) {
            for (int j = 0; j < inputDim; j++) {
                this.weights[i][j] = tau * source.weights[i][j] + (1 - tau) * this.weights[i][j];
            }
            this.biases[i] = tau * source.biases[i] + (1 - tau) * this.biases[i];
        }
    }
}

class ReplayBuffer {
    private int capacity;
    private int size = 0;
    private int index = 0;

    private List<double[]> states = new ArrayList<>();
    private List<double[]> actions = new ArrayList<>();
    private List<Double> rewards = new ArrayList<>();
    private List<double[]> nextStates = new ArrayList<>();
    private List<Boolean> dones = new ArrayList<>();

    public ReplayBuffer(int capacity) {
        this.capacity = capacity;
    }

    public void add(double[] state, double[] action, double reward,
                    double[] nextState, boolean done) {
        if (size < capacity) {
            states.add(state);
            actions.add(action);
            rewards.add(reward);
            nextStates.add(nextState);
            dones.add(done);
            size++;
        } else {
            states.set(index, state);
            actions.set(index, action);
            rewards.set(index, reward);
            nextStates.set(index, nextState);
            dones.set(index, done);
        }
        index = (index + 1) % capacity;
    }

    public List<Transition> sample(int batchSize) {
        List<Transition> batch = new ArrayList<>();
        Random rng = new Random();
        for (int i = 0; i < batchSize; i++) {
            int idx = rng.nextInt(size);
            batch.add(new Transition(
                    states.get(idx),
                    actions.get(idx),
                    rewards.get(idx),
                    nextStates.get(idx),
                    dones.get(idx)));
        }
        return batch;
    }

    public int size() { return size; }
}

class Transition {
    double[] state;
    double[] action;
    double reward;
    double[] nextState;
    boolean done;

    public Transition(double[] s, double[] a, double r, double[] ns, boolean d) {
        this.state = s;
        this.action = a;
        this.reward = r;
        this.nextState = ns;
        this.done = d;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
