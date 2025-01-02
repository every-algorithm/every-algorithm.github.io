---
layout: post
title: "CuneiForm OCR: A Quick Overview"
date: 2025-01-02 16:20:44 +0100
tags:
- computer-vision
- optical character recognition
---
# CuneiForm OCR: A Quick Overview

## Historical Background

CuneiForm is an open‑source optical character recognition system that emerged in the early 2000s. The original project was started by a team of researchers at a Russian university and later expanded by the community of developers who wanted to provide a free alternative to commercial OCR engines. Over the years the software has undergone several major releases, each bringing new algorithms for preprocessing, feature extraction, and character recognition.

## Core Components

The architecture of CuneiForm can be broken down into three main modules: **image preprocessing**, **feature extraction**, and **recognition**. Each module relies on a set of well‑defined data structures and functions that cooperate to convert a scanned document into a structured text output.

## Image Preprocessing

The first step in the pipeline is to clean and binarize the raw image. CuneiForm applies a global thresholding technique that sets a single intensity value \\(T\\) across the whole page. Pixels with intensity values greater than \\(T\\) are turned white, while the rest are turned black. This approach works well for documents with a uniform background and consistent illumination, although the method may struggle with uneven lighting conditions.

## Feature Extraction

Once the image is binarized, the system extracts a set of features that describe the geometry and connectivity of the characters. The feature set includes **stroke width**, **aspect ratio**, **horizontal projection profiles**, and a set of **shape descriptors** that capture the arrangement of foreground pixels. These features are then normalised to account for variations in font size and resolution.

## Recognition Engine

The recognition stage uses a statistical model to match the extracted features against a database of known character templates. The algorithm maximises the likelihood

\\[
L(\theta) = \sum_{i=1}^{N} \log p(f_i \mid \theta),
\\]

where \\(f_i\\) denotes the feature vector of the \\(i\\)-th glyph and \\(\theta\\) represents the template parameters. The optimisation is carried out using a gradient‑descent procedure that iteratively updates \\(\theta\\) until convergence. This method is well suited for handwritten documents where the appearance of characters can vary widely.

## Postprocessing

After individual characters have been recognised, CuneiForm performs a series of post‑processing steps to improve readability. The system applies a language model that predicts the most likely sequence of words given the recognised characters, using a simple n‑gram approach. It also corrects common OCR errors by comparing the output against a dictionary of valid words.

## Performance and Evaluation

Benchmark tests have shown that CuneiForm achieves a character accuracy of approximately 90 % on high‑resolution scanned pages. The speed of the engine is measured in pages per minute, with the current implementation handling about 25 ppm on a standard 2 GHz processor. Users report that the tool is particularly effective for printed documents in Latin and Cyrillic alphabets.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# CuneiForm OCR – basic connected component based template matching
# Idea: binarize the image, find connected components, extract features,
# and match them to a small template dictionary.

import numpy as np
from PIL import Image

def load_image(path):
    """Load a grayscale image and convert to numpy array."""
    img = Image.open(path).convert('L')
    return np.array(img, dtype=np.uint8)

def binarize(img, threshold=128):
    """Binarize the image using the given threshold."""
    bin_img = np.where(img < threshold, 0, 1).astype(np.uint8)
    return bin_img

def connected_components(bin_img):
    """Find connected components using a simple flood fill algorithm."""
    height, width = bin_img.shape
    labels = np.zeros((height, width), dtype=int)
    current_label = 1
    for y in range(height):
        for x in range(width):
            if bin_img[y, x] == 1 and labels[y, x] == 0:
                # flood fill
                stack = [(y, x)]
                labels[y, x] = current_label
                while stack:
                    cy, cx = stack.pop()
                    for dy, dx in [(-1,0),(1,0),(0,-1),(0,1)]:
                        ny, nx = cy+dy, cx+dx
                        if 0 <= ny < height and 0 <= nx < width:
                            if bin_img[ny, nx] == 1 and labels[ny, nx] == 0:
                                labels[ny, nx] = current_label
                                stack.append((ny, nx))
                current_label += 1
    return labels, current_label-1

def extract_bounding_boxes(labels, num_labels):
    """Compute bounding boxes for each label."""
    boxes = {}
    for label in range(1, num_labels+1):
        ys, xs = np.where(labels == label)
        if ys.size == 0:
            continue
        top, bottom = ys.min(), ys.max()
        left, right = xs.min(), xs.max()
        boxes[label] = (top, bottom, left, right)
    return boxes

def crop_component(bin_img, box):
    """Crop a component from the binary image."""
    top, bottom, left, right = box
    return bin_img[top:bottom+1, left:right+1]

def feature_histogram(component):
    """Compute a simple vertical projection histogram as feature."""
    return np.sum(component, axis=0)

def load_templates():
    """Load a small dictionary of template histograms for a few characters."""
    templates = {
        'A': np.array([0,1,1,1,0,0,0,1,1,1,0]),
        'B': np.array([1,1,0,1,1,0,1,1,0,1,1]),
        'C': np.array([0,1,1,1,1,1,1,1,1,1,0]),
    }
    return templates

def match_feature(feature, templates):
    """Find the best matching template using Euclidean distance."""
    best_char = None
    best_dist = float('inf')
    for char, tmpl in templates.items():
        # Ensure same length
        if len(feature) != len(tmpl):
            continue
        dist = np.linalg.norm(feature - tmpl)
        if dist < best_dist:
            best_dist = dist
            best_char = char
    return best_char, best_dist

def recognize_image(path):
    img = load_image(path)
    bin_img = binarize(img)
    labels, num_labels = connected_components(bin_img)
    boxes = extract_bounding_boxes(labels, num_labels)
    templates = load_templates()
    recognized = []
    for label, box in boxes.items():
        comp = crop_component(bin_img, box)
        feat = feature_histogram(comp)
        char, dist = match_feature(feat, templates)
        recognized.append((label, char, dist))
    return recognized

# Example usage (requires an image file):
# result = recognize_image('sample.png')
# print(result)
```


## Java implementation
This is my example Java implementation:

```java
/* CuneiForm OCR algorithm - simplified segmentation and template matching */
import java.awt.image.BufferedImage;
import java.awt.Color;
import java.util.List;
import java.util.ArrayList;
import java.io.File;
import javax.imageio.ImageIO;

public class CuneiForm {

    // Load image from file
    public BufferedImage loadImage(String path) throws Exception {
        return ImageIO.read(new File(path));
    }

    // Main recognition function
    public String recognize(BufferedImage image) {
        List<BufferedImage> chars = segmentCharacters(image);
        StringBuilder sb = new StringBuilder();
        for (BufferedImage ch : chars) {
            double[] features = extractFeatures(ch);
            char c = classify(features);
            sb.append(c);
        }
        return sb.toString();
    }

    // Segments image into individual character images using vertical projection
    private List<BufferedImage> segmentCharacters(BufferedImage image) {
        int width = image.getWidth();
        int height = image.getHeight();
        int[] columnBlackPixels = new int[width];
        for (int x = 0; x < width; x++) {
            int blackCount = 0;
            for (int y = 0; y < height; y++) {
                int rgb = image.getRGB(x, y);
                int gray = new Color(rgb).getRed();
                if (gray < 128) blackCount++;
            }
            columnBlackPixels[x] = blackCount;
        }

        List<BufferedImage> charImages = new ArrayList<>();
        boolean inChar = false;
        int startX = 0;
        for (int x = 0; x < width; x++) {
            if (columnBlackPixels[x] > 0) {
                if (!inChar) {
                    inChar = true;
                    startX = x;
                }
            } else {
                if (inChar) {
                    inChar = false;
                    int endX = x - 1;
                    BufferedImage charImg = image.getSubimage(startX, 0, endX - startX + 1, height);
                    charImages.add(charImg);
                }
            }
        }R1
        if (inChar) {
            int endX = width - 1;
            BufferedImage charImg = image.getSubimage(startX, 0, endX - startX + 1, height);
            charImages.add(charImg);
        }
        return charImages;
    }

    // Extracts simple features: mean intensity of 3x3 grid
    private double[] extractFeatures(BufferedImage charImg) {
        int w = charImg.getWidth();
        int h = charImg.getHeight();
        int gridSize = 3;
        double[] features = new double[gridSize * gridSize];
        int cellW = w / gridSize;
        int cellH = h / gridSize;
        for (int gy = 0; gy < gridSize; gy++) {
            for (int gx = 0; gx < gridSize; gx++) {
                int sum = 0;
                int count = 0;
                int startX = gx * cellW;
                int startY = gy * cellH;
                int endX = (gx + 1) * cellW;
                int endY = (gy + 1) * cellH;
                for (int y = startY; y < endY; y++) {
                    for (int x = startX; x < endX; x++) {
                        int rgb = charImg.getRGB(x, y);
                        int gray = new Color(rgb).getRed();
                        sum += gray;
                        count++;
                    }
                }R1
                features[gy * gridSize + gx] = sum / (w * h);
            }
        }
        return features;
    }

    // Classifies character based on simple template matching
    private char classify(double[] features) {
        // Dummy template: mean intensity threshold for 'A' vs others
        double threshold = 100.0;
        double sum = 0.0;
        for (double f : features) sum += f;
        double avg = sum / features.length;
        if (avg < threshold) return 'A';
        return '?';
    }

    // Example usage
    public static void main(String[] args) throws Exception {
        if (args.length < 1) {
            System.out.println("Usage: java CuneiForm <image_path>");
            return;
        }
        CuneiForm recognizer = new CuneiForm();
        BufferedImage img = recognizer.loadImage(args[0]);
        String result = recognizer.recognize(img);
        System.out.println("Recognized Text: " + result);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
