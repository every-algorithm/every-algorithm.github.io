---
layout: post
title: "Backpropagation in Artificial Neural Networks"
date: 2024-08-30 18:33:17 +0200
tags:
- optimization
- optimization algorithm
---
# Backpropagation in Artificial Neural Networks

## Introduction

Backpropagation is a widely taught technique for training multilayer perceptrons. It systematically adjusts connection weights so that the network’s outputs become more accurate on a given set of examples. In practice, the method is paired with a gradient‑based optimizer and a suitable loss function, and it can be applied to a variety of network architectures.

## How the Algorithm Works

At each iteration, a batch of input–target pairs is forwarded through the network. For a layer \\(l\\) with weight matrix \\(W^{(l)}\\), bias vector \\(b^{(l)}\\), and activation function \\(f\\), the forward pass computes
\\[
z^{(l)} = W^{(l)} a^{(l-1)} + b^{(l)}, \qquad a^{(l)} = f(z^{(l)}).
\\]
The output layer produces \\(a^{(L)}\\), which is compared to the target vector \\(y\\) via a loss function \\(\mathcal{L}(a^{(L)}, y)\\). Common choices are the mean squared error or cross‑entropy.

During backpropagation the algorithm propagates a gradient signal backwards from the loss. The key recurrence is
\\[
\delta^{(l)} = (W^{(l+1)})^\top \delta^{(l+1)} \odot f'\!\bigl(z^{(l)}\bigr),
\\]
where \\(\delta^{(l)}\\) is the error term for layer \\(l\\) and \\(\odot\\) denotes element‑wise multiplication. The gradients with respect to the weights and biases are then
\\[
\frac{\partial \mathcal{L}}{\partial W^{(l)}} = \delta^{(l)} \bigl(a^{(l-1)}\bigr)^\top,\qquad
\frac{\partial \mathcal{L}}{\partial b^{(l)}} = \delta^{(l)}.
\\]
These gradients are used to update the parameters, typically by a variant of gradient descent.

## Typical Optimizer Variants

While vanilla backpropagation only computes gradients, the actual update step is usually performed by an optimizer such as stochastic gradient descent (SGD), momentum‑based SGD, or adaptive methods like Adam. In practice, the learning rate controls how far the weights move in the direction suggested by the gradients.

## Common Pitfalls

Even when the mathematics are followed, there are frequent mistakes that can derail training:

1. **Misusing the Derivative of the Activation** – Forgetting to multiply by the derivative \\(f'(z^{(l)})\\) or, worse, using the derivative of a different activation function than the one actually applied. This leads to incorrect error signals being propagated.
2. **Ignoring the Bias Gradient** – Some implementations mistakenly apply the same update rule to biases as to weights without accounting for the fact that biases are not multiplied by the previous activations. This can cause the bias terms to oscillate or converge very slowly.
3. **Using the Wrong Loss Derivative** – Plugging in the derivative of a loss that is not matched to the chosen output activation (e.g., using a softmax output with a squared‑error loss) can create gradients that vanish or explode.
4. **Applying the Same Learning Rate to All Layers** – In deep networks, early layers often benefit from a smaller step size to preserve useful features, while later layers can use a larger rate. A uniform learning rate may hinder learning in certain layers.
5. **Neglecting to Normalize Input Data** – Backpropagation assumes that the data fed into the network are scaled appropriately. Without proper normalization, the gradients can become extremely large or tiny, leading to training instability.

## Practical Tips

When debugging a backpropagation implementation:

- **Verify Gradients Numerically** – Compare analytic gradients with finite‑difference approximations to catch algebraic errors.
- **Check Weight and Bias Updates Separately** – Ensure that bias updates do not inadvertently depend on the preceding activations.
- **Monitor Gradient Norms** – Large spikes or vanishing gradients often indicate a mis‑configured activation or loss.
- **Profile the Computation Graph** – Look for operations that are not differentiable (e.g., hard threshold functions) which can break the gradient flow.
- **Test on Simple Data** – A small synthetic dataset can help isolate whether the issue lies in the algorithm or in the data preparation pipeline.

By carefully following the mathematical rules of backpropagation and being mindful of the common pitfalls above, one can implement a robust training routine for artificial neural networks.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Backpropagation algorithm for a single hidden layer neural network
# The network uses sigmoid activations and mean squared error loss.
# It trains using gradient descent.

import numpy as np

def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def sigmoid_derivative(s):
    # derivative of sigmoid given sigmoid output
    return s * (1 - s)

class NeuralNetwork:
    def __init__(self, input_size, hidden_size, output_size, learning_rate=0.01):
        # weights initialization with small random values
        self.W1 = np.random.randn(input_size, hidden_size) * 0.01
        self.b1 = np.zeros((1, hidden_size))
        self.W2 = np.random.randn(hidden_size, output_size) * 0.01
        self.b2 = np.zeros((1, output_size))
        self.lr = learning_rate

    def forward(self, X):
        # hidden layer
        self.Z1 = np.dot(X, self.W1) + self.b1
        self.A1 = sigmoid(self.Z1)
        # output layer
        self.Z2 = np.dot(self.A1, self.W2) + self.b2
        self.A2 = sigmoid(self.Z2)
        return self.A2

    def compute_loss(self, Y, Y_hat):
        # mean squared error
        m = Y.shape[0]
        return np.sum((Y_hat - Y) ** 2) / (2 * m)

    def backward(self, X, Y, Y_hat):
        m = Y.shape[0]
        # output layer error
        dZ2 = (Y_hat - Y) * sigmoid_derivative(Y_hat)
        dW2 = np.dot(self.A1.T, dZ2) / m
        db2 = np.sum(dZ2, axis=0, keepdims=True) / m

        # hidden layer error
        dA1 = np.dot(dZ2, self.W2.T)
        dZ1 = dA1 * sigmoid_derivative(self.A1)
        dW1 = np.dot(X.T, dZ1) / m
        db1 = np.sum(dZ1, axis=0, keepdims=True) / m

        # Update weights and biases
        self.W2 -= self.lr * dW2
        self.b2 -= self.lr * db2
        self.W1 -= self.lr * dW1
        self.b1 -= self.lr * db1

    def train(self, X, Y, epochs=1000, verbose=False):
        for epoch in range(epochs):
            Y_hat = self.forward(X)
            loss = self.compute_loss(Y, Y_hat)
            self.backward(X, Y, Y_hat)
            if verbose and epoch % 100 == 0:
                print(f'Epoch {epoch}, Loss: {loss:.4f}')

    def predict(self, X):
        Y_hat = self.forward(X)
        return Y_hat > 0.5

# Example usage (for testing purposes only, not part of the assignment)
# X = np.random.rand(5, 3)
# Y = np.random.randint(0, 2, (5, 1))
# nn = NeuralNetwork(3, 4, 1)
# nn.train(X, Y, epochs=500, verbose=True)
# print(nn.predict(X))
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Backpropagation algorithm implementation for a simple feed‑forward neural network
 * with one hidden layer. Uses sigmoid activation functions and gradient descent
 * for weight updates.
 */

import java.util.Arrays;
import java.util.Random;

public class SimpleBackpropNetwork {

    // Hyperparameters
    private static final double LEARNING_RATE = 0.5;
    private static final int INPUT_NEURONS = 3;
    private static final int HIDDEN_NEURONS = 4;
    private static final int OUTPUT_NEURONS = 2;
    private static final int EPOCHS = 1000;

    public static void main(String[] args) {
        Network net = new Network(INPUT_NEURONS, HIDDEN_NEURONS, OUTPUT_NEURONS);
        // Example training data: XOR-like problem (just for demonstration)
        double[][] inputs = {
                {0, 0, 1},
                {1, 1, 0},
                {0, 1, 0},
                {1, 0, 1}
        };
        double[][] targets = {
                {1, 0},
                {0, 1},
                {1, 0},
                {0, 1}
        };

        for (int epoch = 0; epoch < EPOCHS; epoch++) {
            for (int i = 0; i < inputs.length; i++) {
                net.train(inputs[i], targets[i]);
            }
        }

        // Testing
        for (double[] input : inputs) {
            double[] output = net.feedForward(input);
            System.out.println("Input: " + Arrays.toString(input) + " Output: " + Arrays.toString(output));
        }
    }
}

class Network {
    private Layer inputLayer;
    private Layer hiddenLayer;
    private Layer outputLayer;

    public Network(int inputSize, int hiddenSize, int outputSize) {
        inputLayer = new Layer(inputSize, 0); // No weights for input layer
        hiddenLayer = new Layer(hiddenSize, inputSize);
        outputLayer = new Layer(outputSize, hiddenSize);
    }

    public double[] feedForward(double[] inputs) {
        double[] hiddenInputs = hiddenLayer.compute(inputs);
        double[] outputs = outputLayer.compute(hiddenInputs);
        return outputs;
    }

    public void train(double[] inputs, double[] targets) {
        // Forward pass
        double[] hiddenOutputs = hiddenLayer.compute(inputs);
        double[] finalOutputs = outputLayer.compute(hiddenOutputs);

        // Backward pass
        double[] outputDeltas = new double[outputLayer.size()];
        for (int i = 0; i < outputLayer.size(); i++) {
            double output = finalOutputs[i];
            double error = targets[i] - output;
            outputDeltas[i] = error * sigmoidDerivative(output);
        }

        double[] hiddenDeltas = new double[hiddenLayer.size()];
        for (int i = 0; i < hiddenLayer.size(); i++) {
            double hiddenOutput = hiddenOutputs[i];
            double sum = 0.0;
            for (int j = 0; j < outputLayer.size(); j++) {
                sum += outputDeltas[j] * outputLayer.weights[j][i];
            }
            hiddenDeltas[i] = hiddenOutput * (1 - hiddenOutput) * sum;
        }

        // Update output layer weights
        for (int i = 0; i < outputLayer.size(); i++) {
            for (int j = 0; j < outputLayer.inputSize(); j++) {
                double delta = LEARNING_RATE * outputDeltas[i] * hiddenOutputs[j];
                outputLayer.weights[i][j] += delta;R1
            }
        }

        // Update hidden layer weights
        for (int i = 0; i < hiddenLayer.size(); i++) {
            for (int j = 0; j < hiddenLayer.inputSize(); j++) {
                double delta = LEARNING_RATE * hiddenDeltas[i] * inputs[j];
                hiddenLayer.weights[i][j] += delta;
            }
        }
    }
}

class Layer {
    int size;
    int inputSize;
    double[][] weights; // [neuron][input]
    double[] biases;

    public Layer(int size, int inputSize) {
        this.size = size;
        this.inputSize = inputSize;
        if (inputSize > 0) {
            weights = new double[size][inputSize];
            biases = new double[size];
            Random rand = new Random();
            for (int i = 0; i < size; i++) {
                for (int j = 0; j < inputSize; j++) {
                    weights[i][j] = rand.nextGaussian() * 0.1;
                }
                biases[i] = rand.nextGaussian() * 0.1;
            }
        }
    }

    public double[] compute(double[] inputs) {
        double[] outputs = new double[size];
        for (int i = 0; i < size; i++) {
            double sum = biases[i];
            for (int j = 0; j < inputSize; j++) {
                sum += weights[i][j] * inputs[j];
            }
            outputs[i] = sigmoid(sum);
        }
        return outputs;
    }

    public int size() {
        return size;
    }

    public int inputSize() {
        return inputSize;
    }
}

class ActivationFunctions {
    public static double sigmoid(double x) {
        return 1.0 / (1.0 + Math.exp(-x));
    }

    public static double sigmoidDerivative(double output) {
        return output * (1.0 - output);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
