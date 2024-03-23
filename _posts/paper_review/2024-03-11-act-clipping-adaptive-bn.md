---
layout: default
title: >
    Data-Free Quantization with Accurate Activation Clipping and Adaptive Batch Normalization
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240311
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

 **Keywords: #Activation #Batch Normalization**

---

## 0. Abstract
- Data-free quantization: Compresses NN to low bit-width without access to original training data. 
- Most data-free quantization cause severe performance degradation due to (1) inaccurate activation clipping range and (2) quantization error. 
- Proposal
    - Accurate activation clipping (AAC)
    - Adaptive batch normalization (ABN)

## 1. Introduction
- QAT: Retrain the quantized model and consumes lots of computation resources
- PTQ: Directly quantize the pre-trained FP model and use part of the training data to calibrate it. 
- Both require real data in the quantizaiton process → However, **the training dataset is not available** in many practical scenarios. 

## 3. Method
### 3.1. Preliminaries
- We use uniform quantization, where given clip range is $[l,u]$ for weight or activation.
- Two problems of data-free quantization 
    1. The range of weights is fixed by training, however the clip range for activations depends on the specific input and is unknown. 
    2. The statistics of BN layers depend on the input and feature maps of the network, and have been fixed in the model during training. 

### 3.2. Accurate Activation Clipping 
- How to determine the clipping range of activations
    - Generate synthetic data and conduct forward propagation with it. *The peak value of activations is stored as the clipping range parameter.* (using `clamp(max=??)`)
    - Synthetic data is generated using distribution of BN layers → Not a good prediction for activation range.
    - Furthermore, the peaks are not directly related to the distribution of the data. 
- So what exactly determines the activation clipping range? (??) 
- ReLU: The lower bound $l$ is zero, while the upper bound $u$ depends on the maximum value of the feature map. `Learning deep features for discriminative localization` study shows that high response is related to significant features. Therefore, synthetic data should make the network be highly responsive. 
- AAC: (??)

### 3.3. Adaptive Batch Normalization 
- `Data-free quantization through weight equalization and bias correction`: When CONV layer is quantized, the output featuremap distribution will have a certain offset compared to the original model. 