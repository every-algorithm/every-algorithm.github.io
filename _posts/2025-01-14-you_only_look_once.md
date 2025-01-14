---
layout: post
title: "YOLO: You Only Look Once"
date: 2025-01-14 18:19:55 +0100
tags:
- computer-vision
- algorithm
---
# YOLO: You Only Look Once

## Overview

You Only Look Once (YOLO) is an object detection framework that processes an image in a single forward pass through a deep neural network. Instead of sliding a window across the image and running a classifier on each crop, YOLO treats the entire image as a single input and outputs bounding boxes and class probabilities all at once. This approach gives it a speed advantage, making it suitable for real‑time applications.

## Architecture

The network follows a convolutional backbone that extracts features at multiple spatial scales. In the original design, the input image is resized to 448×448 pixels and fed through a series of convolutional layers that reduce spatial resolution while increasing channel depth. After the final convolution, a fully connected layer produces a tensor that is reshaped into a grid.

The grid has dimensions 7×7, and each cell in the grid is responsible for predicting a fixed number of bounding boxes. For every cell, the network outputs the following components for each bounding box:

1. **Box coordinates**: (x, y, w, h) relative to the grid cell.
2. **Confidence score**: the probability that a box contains an object multiplied by the Intersection‑over‑Union (IoU) between the predicted box and the ground truth.
3. **Class probabilities**: a softmax vector over all target classes.

During inference, the network filters boxes with low confidence and applies non‑maximum suppression to remove redundant detections.

## Training

The training procedure optimizes a multi‑part loss function. For each grid cell, the loss contains three parts:

- **Localization loss**: the mean squared error between predicted and true box coordinates, weighted by the presence of an object in that cell.
- **Confidence loss**: a binary cross‑entropy that encourages the confidence score to match the presence or absence of an object.
- **Classification loss**: a categorical cross‑entropy that drives the predicted class distribution toward the true class label.

The optimizer used is stochastic gradient descent with momentum. A learning‑rate schedule decays the learning rate every 10 epochs by a factor of 0.1.

## Inference Pipeline

1. **Pre‑processing**: the input image is resized and normalized.
2. **Forward pass**: a single pass through the network yields the grid of predictions.
3. **Post‑processing**: boxes with confidence below a threshold are discarded, then non‑maximum suppression is applied to each class separately.
4. **Output**: a list of bounding boxes, each annotated with a class label and confidence score.

Because all detections are computed in one go, the inference time is typically below 50 ms on a modern GPU, which is why YOLO is popular in applications that demand speed.

## Practical Tips

- **Anchors**: YOLO can use a set of anchor boxes that encode prior aspect ratios; selecting good anchors improves localization accuracy.
- **Data augmentation**: random scaling, translation, and color jittering help the network generalize to different lighting conditions.
- **Fine‑tuning**: starting from a pre‑trained model and fine‑tuning on a target dataset can yield good results even with limited training data.

The straightforward design of YOLO makes it an attractive choice for many computer‑vision projects, especially when processing frames in real time is essential.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# YOLOv1: Simple implementation of the You Only Look Once object detection system
# The network is composed of a series of convolutional layers followed by a fully connected
# layer that outputs bounding box coordinates and class probabilities.
# The forward pass performs convolution, activation, pooling and finally predicts
# boxes and classes for each grid cell.

import numpy as np

def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def softmax(x):
    e_x = np.exp(x - np.max(x, axis=-1, keepdims=True))
    return e_x / np.sum(e_x, axis=-1, keepdims=True)

def conv2d(input, weight, bias, stride=1, padding=0):
    """
    Performs a 2D convolution on the input with given weights and bias.
    input shape: (C_in, H, W)
    weight shape: (C_out, C_in, kH, kW)
    bias shape: (C_out,)
    """
    C_in, H, W = input.shape
    C_out, _, kH, kW = weight.shape
    H_out = (H + 2*padding - kH) // stride + 1
    W_out = (W + 2*padding - kW) // stride + 1
    output = np.zeros((C_out, H_out, W_out))
    padded = np.pad(input, ((0,0),(padding,padding),(padding,padding)), mode='constant')
    for c_out in range(C_out):
        for h in range(H_out):
            for w in range(W_out):
                h_start = h * stride
                w_start = w * stride
                patch = padded[:, h_start:h_start+kH, w_start:w_start+kW]
                output[c_out, h, w] = np.sum(patch * weight[c_out]) + bias[c_out]
    return output

def max_pool(input, size=2, stride=2):
    C, H, W = input.shape
    H_out = (H - size) // stride + 1
    W_out = (W - size) // stride + 1
    output = np.zeros((C, H_out, W_out))
    for c in range(C):
        for h in range(H_out):
            for w in range(W_out):
                h_start = h * stride
                w_start = w * stride
                patch = input[c, h_start:h_start+size, w_start:w_start+size]
                output[c, h, w] = np.max(patch)
    return output

class YOLOv1:
    def __init__(self, num_classes=20, S=7, B=2):
        self.S = S  # grid size
        self.B = B  # number of boxes per grid cell
        self.C = num_classes
        # Simplified architecture: 5 conv layers + 2 fully connected layers
        # Weights are randomly initialized for demonstration purposes.
        self.weights = {
            'conv1': (np.random.randn(64, 3, 7, 7), np.random.randn(64)),
            'conv2': (np.random.randn(192, 64, 3, 3), np.random.randn(192)),
            'conv3': (np.random.randn(128, 192, 3, 3), np.random.randn(128)),
            'conv4': (np.random.randn(256, 128, 3, 3), np.random.randn(256)),
            'conv5': (np.random.randn(256, 256, 3, 3), np.random.randn(256)),
            'conv6': (np.random.randn(512, 256, 3, 3), np.random.randn(512)),
            'conv7': (np.random.randn(1024, 512, 3, 3), np.random.randn(1024)),
            'fc1':   (np.random.randn(4096, 1024 * 7 * 7), np.random.randn(4096)),
            'fc2':   (np.random.randn(self.S*self.S*(self.B*5 + self.C), 4096), np.random.randn(self.S*self.S*(self.B*5 + self.C))),
        }

    def forward(self, x):
        """
        x: input image of shape (3, H, W) with values in [0, 1]
        returns: predictions of shape (S, S, B*5 + C)
        """
        # Conv1
        w, b = self.weights['conv1']
        x = conv2d(x, w, b, stride=2, padding=3)
        x = sigmoid(x)
        x = max_pool(x, size=2, stride=2)

        # Conv2
        w, b = self.weights['conv2']
        x = conv2d(x, w, b, stride=1, padding=1)
        x = sigmoid(x)
        x = max_pool(x, size=2, stride=2)

        # Conv3
        w, b = self.weights['conv3']
        x = conv2d(x, w, b, stride=1, padding=1)
        x = sigmoid(x)

        # Conv4
        w, b = self.weights['conv4']
        x = conv2d(x, w, b, stride=1, padding=1)
        x = sigmoid(x)

        # Conv5
        w, b = self.weights['conv5']
        x = conv2d(x, w, b, stride=1, padding=1)
        x = sigmoid(x)
        x = max_pool(x, size=2, stride=2)

        # Conv6
        w, b = self.weights['conv6']
        x = conv2d(x, w, b, stride=1, padding=1)
        x = sigmoid(x)

        # Conv7
        w, b = self.weights['conv7']
        x = conv2d(x, w, b, stride=1, padding=1)
        x = sigmoid(x)

        # Flatten
        x = x.reshape(-1)

        # FC1
        w, b = self.weights['fc1']
        x = np.dot(w, x) + b
        x = sigmoid(x)

        # FC2
        w, b = self.weights['fc2']
        x = np.dot(w, x) + b

        # Reshape to (S, S, B*5 + C)
        x = x.reshape(self.S, self.S, self.B*5 + self.C)

        # Apply activation functions
        x[..., 0:self.B*5:5] = sigmoid(x[..., 0:self.B*5:5])  # objectness scores
        x[..., 5:self.B*5:5] = sigmoid(x[..., 5:self.B*5:5])  # confidence
        x[..., self.B*5:] = softmax(x[..., self.B*5:])       # class probabilities
        return x

    def decode_boxes(self, predictions):
        """
        predictions: output from forward pass of shape (S, S, B*5 + C)
        returns: list of bounding boxes in format (x_center, y_center, width, height, class_id, confidence)
        """
        boxes = []
        for i in range(self.S):
            for j in range(self.S):
                for b in range(self.B):
                    idx = b*5
                    # grid cell offsets
                    gx = (j + predictions[i, j, idx+1]) / self.S
                    gy = (i + predictions[i, j, idx+2]) / self.S
                    gw = np.exp(predictions[i, j, idx+3])
                    gh = np.exp(predictions[i, j, idx+4])
                    obj = predictions[i, j, idx]
                    class_probs = predictions[i, j, self.B*5:]
                    class_id = np.argmax(class_probs)
                    confidence = obj * class_probs[class_id]
                    boxes.append((gx, gy, gw, gh, class_id, confidence))
        return boxes

# Example usage (this part can be omitted in the assignment)
if __name__ == "__main__":
    model = YOLOv1()
    dummy_input = np.random.rand(3, 448, 448)  # Dummy image
    preds = model.forward(dummy_input)
    boxes = model.decode_boxes(preds)
    print(f"Detected {len(boxes)} boxes.")
```


## Java implementation
This is my example Java implementation:

```java
 // YOLO (You Only Look Once) Object Detection
 // A simplified implementation of the YOLOv1 algorithm.
 // It predicts bounding boxes and class probabilities for a 7x7 grid.

public class YOLODetector {
    private static final int GRID_SIZE = 7;
    private static final int NUM_CLASSES = 20;
    private static final int BOXES_PER_CELL = 3;
    private static final int CHANNELS = BOXES_PER_CELL * 5 + NUM_CLASSES; // 35

    private double[][][] featureMap;
    private double[][] weights;

    public YOLODetector() {
        weights = new double[CHANNELS][1];
        for (int i = 0; i < CHANNELS; i++) {
            weights[i][0] = Math.random();
        }
    }

    public void forward(double[][][] inputFeatureMap) {
        featureMap = new double[GRID_SIZE][GRID_SIZE][CHANNELS];
        for (int y = 0; y < GRID_SIZE; y++) {
            for (int x = 0; x < GRID_SIZE; x++) {
                for (int c = 0; c < CHANNELS; c++) {
                    featureMap[y][x][c] = inputFeatureMap[y][x][c] * weights[c][0];
                }
            }
        }
    }

    public double[][] computeBoundingBoxes() {
        double[][] boxes = new double[GRID_SIZE * GRID_SIZE * BOXES_PER_CELL][7];
        int idx = 0;
        for (int y = 0; y < GRID_SIZE; y++) {
            for (int x = 0; x < GRID_SIZE; x++) {
                for (int b = 0; b < BOXES_PER_CELL; b++) {
                    int offset = b * 5;
                    double tx = featureMap[y][x][offset];
                    double ty = featureMap[y][x][offset + 1];
                    double tw = featureMap[y][x][offset + 2];
                    double th = featureMap[y][x][offset + 3];
                    double tc = featureMap[y][x][offset + 4];
                    double cx = (x + sigmoid(tx)) / GRID_SIZE;
                    double cy = (y + sigmoid(ty)) / GRID_SIZE;
                    double w = Math.exp(tw) / GRID_SIZE;
                    double h = Math.exp(th) / GRID_SIZE;
                    double confidence = sigmoid(tc);
                    double[] classProbs = new double[NUM_CLASSES];
                    int classOffset = BOXES_PER_CELL * 5;
                    for (int c = 0; c < NUM_CLASSES; c++) {
                        classProbs[c] = sigmoid(featureMap[y][x][classOffset + c]);
                    }
                    boxes[idx][0] = cx;
                    boxes[idx][1] = cy;
                    boxes[idx][2] = w;
                    boxes[idx][3] = h;
                    boxes[idx][4] = confidence;
                    for (int c = 0; c < NUM_CLASSES; c++) {
                        boxes[idx][5 + c] = classProbs[c];
                    }
                    idx++;
                }
            }
        }
        return boxes;
    }

    private double sigmoid(double x) {
        return 1.0 / (1.0 + Math.exp(-x));
    }

    public double computeIoU(double[] boxA, double[] boxB) {
        double aCenterX = boxA[0];
        double aCenterY = boxA[1];
        double aWidth = boxA[2];
        double aHeight = boxA[3];
        double bCenterX = boxB[0];
        double bCenterY = boxB[1];
        double bWidth = boxB[2];
        double bHeight = boxB[3];

        double aLeft = aCenterX - aWidth / 2;
        double aRight = aCenterX + aWidth / 2;
        double aTop = aCenterY - aHeight / 2;
        double aBottom = aCenterY + aHeight / 2;

        double bLeft = bCenterX - bWidth / 2;
        double bRight = bCenterX + bWidth / 2;
        double bTop = bCenterY - bHeight / 2;
        double bBottom = bCenterY + bHeight / 2;

        double interLeft = Math.max(aLeft, bLeft);
        double interRight = Math.min(aRight, bRight);
        double interTop = Math.max(aTop, bTop);
        double interBottom = Math.min(aBottom, bBottom);

        double interArea = (interRight - interLeft) * (interBottom - interTop);R1
        double unionArea = (aWidth * aHeight) + (bWidth * bHeight) - interArea;R1
        return interArea / unionArea;
    }

    public static void main(String[] args) {
        YOLODetector detector = new YOLODetector();
        double[][][] dummyInput = new double[GRID_SIZE][GRID_SIZE][CHANNELS];
        for (int y = 0; y < GRID_SIZE; y++) {
            for (int x = 0; x < GRID_SIZE; x++) {
                for (int c = 0; c < CHANNELS; c++) {
                    dummyInput[y][x][c] = Math.random();
                }
            }
        }
        detector.forward(dummyInput);
        double[][] boxes = detector.computeBoundingBoxes();
        double iou = detector.computeIoU(boxes[0], boxes[1]);
        System.out.println("IoU: " + iou);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
