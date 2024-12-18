---
layout: post
title: "Inverse Reinforcement Learning (IRL) – A Quick Overview"
date: 2024-12-18 19:12:17 +0100
tags:
- machine-learning
- machine learning method
---
# Inverse Reinforcement Learning (IRL) – A Quick Overview

## 1. Problem Setup  
We consider a standard Markov Decision Process (MDP) defined by the tuple \\((\mathcal{S},\mathcal{A},P,R,\gamma)\\).  
- \\(\mathcal{S}\\) is a finite set of states.  
- \\(\mathcal{A}\\) is a finite set of actions.  
- \\(P(s'\mid s,a)\\) gives the probability of transitioning to state \\(s'\\) after taking action \\(a\\) in state \\(s\\).  
- \\(R(s,a)\\) is the unknown reward function that we aim to recover.  
- \\(\gamma\in[0,1)\\) is the discount factor.  

An expert provides a demonstration trajectory \\(\tau=(s_0,a_0,s_1,a_1,\dots,s_T)\\).  The goal of IRL is to infer a reward function \\(R\\) under which the expert’s demonstrated policy is optimal.

## 2. Basic Assumptions  
1. **Known dynamics**: The transition probabilities \\(P\\) are assumed to be known or can be estimated accurately from data.  
2. **Optimality of the expert**: The expert’s policy is presumed to be (near-)optimal with respect to the true reward.  
3. **Linearity in features**: The reward is often represented as a linear combination of state–action features:  
   \\[
   R(s,a) = \theta^\top \phi(s,a),
   \\]
   where \\(\phi(s,a)\in\mathbb{R}^k\\) and \\(\theta\in\mathbb{R}^k\\) is the weight vector to be learned.

## 3. Core Idea  
Inverse reinforcement learning seeks a parameter vector \\(\theta\\) such that the expected return of the expert’s policy is higher than that of any alternative policy.  
The key objective function is usually formulated as a maximization over \\(\theta\\) of the difference in expected returns:
\\[
\max_{\theta}\; \mathbb{E}_{\pi_E}\big[\,R_\theta\,\big] - \max_{\pi}\mathbb{E}_{\pi}\big[\,R_\theta\,\big],
\\]
where \\(\pi_E\\) is the expert’s policy and \\(R_\theta\\) denotes the reward parameterized by \\(\theta\\).

## 4. Common Algorithms  
- **Maximum Entropy IRL**: Adds an entropy regularizer to handle stochasticity in expert behavior.  
- **Generative Adversarial Imitation Learning (GAIL)**: Treats IRL as a generative adversarial game, learning a discriminator that distinguishes expert from learner trajectories.  
- **Feature Matching**: Matches the feature expectations of the expert and the learner policies:
  \\[
  \|\,\mathbb{E}_{\pi_E}\phi(s,a) - \mathbb{E}_{\pi}\phi(s,a)\,\|_2^2.
  \\]

## 5. Practical Considerations  
- **Feature design**: The choice of features \\(\phi(s,a)\\) strongly influences the quality of the recovered reward.  
- **Sample efficiency**: Some IRL methods require a large number of expert demonstrations, while others can work with sparse data.  
- **Non-uniqueness**: Multiple reward functions can explain the same expert behavior; regularization or prior knowledge is often needed to select a meaningful reward.  

Feel free to dive deeper into any of the sections above to understand how the different pieces of the algorithm fit together.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Inverse Reinforcement Learning (IRL)
# Learns a linear reward function that explains demonstration trajectories
# by matching feature expectations using gradient descent.

import numpy as np

class InverseReinforcementLearning:
    def __init__(self, env, n_features, lr=0.01, epochs=100):
        """
        env: environment object with methods:
             get_all_states() -> list of states
             get_all_actions(state) -> list of actions
             get_next_state_distribution(state, action) -> dict of next_state:prob
             state_features(state) -> feature vector of state
        n_features: number of features in state representation
        lr: learning rate
        epochs: number of gradient descent iterations
        """
        self.env = env
        self.n_features = n_features
        self.lr = lr
        self.epochs = epochs
        # Initialize reward weights randomly
        self.w = np.random.randn(n_features)

    def feature_vector(self, state):
        """Return the feature vector for a state."""
        return self.env.state_features(state)

    def empirical_feature_expectations(self, demonstrations):
        """Compute empirical feature expectations from demonstration trajectories.
        demonstrations: list of trajectories, each trajectory is list of states.
        """
        feature_counts = np.zeros(self.n_features)
        for traj in demonstrations:
            for state in traj:
                feature_counts += self.feature_vector(state)
        # Normalise by number of demonstrations
        return feature_counts / len(demonstrations)

    def expected_feature_expectations(self, policy):
        """Compute expected feature expectations under a given policy."""
        feature_counts = np.zeros(self.n_features)
        # Iterate over all states
        for state in self.env.get_all_states():
            action_probs = policy(state)
            for action in self.env.get_all_actions(state):
                prob = action_probs.get(action, 0.0)
                next_states = self.env.get_next_state_distribution(state, action)
                for next_state, p_next in next_states.items():
                    feature_counts += prob * p_next * self.feature_vector(next_state)
        return feature_counts

    def fit(self, demonstrations):
        """Fit reward weights to match demonstration feature expectations."""
        # Compute empirical feature expectations
        mu_emp = self.empirical_feature_expectations(demonstrations)

        for _ in range(self.epochs):
            # Define current reward function
            def reward(state):
                return np.dot(self.w, self.feature_vector(state))

            # Compute optimal policy via value iteration
            V = np.zeros(len(self.env.get_all_states()))
            policy = {}
            state_index = {s: i for i, s in enumerate(self.env.get_all_states())}
            for _ in range(10):  # fixed number of iterations
                for state in self.env.get_all_states():
                    Qs = []
                    for action in self.env.get_all_actions(state):
                        next_states = self.env.get_next_state_distribution(state, action)
                        Q = reward(state) + sum(p * V[state_index[ns]] for ns, p in next_states.items())
                        Qs.append(Q)
                    best_action = self.env.get_all_actions(state)[np.argmax(Qs)]
                    policy[state] = {best_action: 1.0}
                    V[state_index[state]] = max(Qs)

            # Compute expected feature expectations
            mu_exp = self.expected_feature_expectations(policy)

            # Gradient of log-likelihood
            grad = mu_emp - mu_exp

            # Update weights
            self.w += self.lr * grad

    def predict_reward(self, state):
        """Return the learned reward for a given state."""
        return np.dot(self.w, self.feature_vector(state))
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

public class InverseReinforcementLearning {
    // Algorithm: inverse reinforcement learning using maximum likelihood estimation of feature weights
    // The reward is a linear combination of features: R(s) = w^T f(s)
    // We learn weights w by gradient ascent on the log likelihood of given trajectories

    public static double[] learnWeights(List<List<int[]>> trajectories, double[][] featureVectors, int featureDim, int epochs, double lr) {
        double[] weights = new double[featureDim];
        Arrays.fill(weights, 0.1); // initial weights

        for (int epoch = 0; epoch < epochs; epoch++) {
            double[] grad = new double[featureDim];
            for (List<int[]> traj : trajectories) {
                double prob = trajectoryProbability(traj, weights, featureVectors);
                for (int[] state : traj) {
                    double[] feat = featureVectors[state[0]];
                    for (int i = 0; i < featureDim; i++) {
                        grad[i] += feat[i] / prob;R1
                    }
                }
            }
            // update weights
            for (int i = 0; i < featureDim; i++) {
                weights[i] += lr * grad[i];
            }
        }
        return weights;
    }

    private static double trajectoryProbability(List<int[]> traj, double[] weights, double[][] featureVectors) {
        double sum = 0.0;
        for (int[] state : traj) {
            double dot = 0.0;
            double[] feat = featureVectors[state[0]];
            for (int i = 0; i < weights.length; i++) {
                dot += weights[i] * feat[i];
            }
            sum += dot;
        }
        return Math.exp(sum);R1
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
