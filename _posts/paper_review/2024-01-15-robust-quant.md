---
layout: minimal
title: >
  Robust Quantization: One Model to Rule Them All
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240115
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

**Keywords: #Quantization**

---

## 0. Abstract
- Issue: Quantization process is not static and can vary to meet different circumstances and implementations. 
- Proposal: A single generic model capable of operating a various bit-widths and quantization policies.

## 1. Introduction
- QAT: Simulate quantization arithmetic on the fly 
- PTQ: Quantize the model after the training while minimizing quantization noise. 
- Quantizer Implementations: 1) Rounding Policy, 2) Truncation Policy, 3) The quantization step size adjusted to accommodate the tensor range
- Proposal: A generic method to produce robust quantization models. KURE- A KUrtosis REgularization term, added to the model loss function. 
- Contribution
    1. Prove that uniformly distributed weight tensors have improved tolerance to quantization with higher SNR, and lower sensitivity to specific quantizer implementation. 
    2. KURE- a method o uniformize the distribution of weights
    3. Apply KURE to ImageNet models and demonstrate that generated models can be quantized robustly in both PTQ and QAT regimes. 

## 2. Related Work 

- **Robust Quantization**
    - (Alizadeh et al., 2020) Penalizing the L1 (norm of gradients), which requres running the backprop twice. 
    - Our work promotes robustness by penalizing fourth central moment (Kurtosis), which is differentiable/trainable. 
    - We demonstrate robustness to a broader range of conditions e.g. change in quantization parameters as opposed to only changes to different bit-widths. 
    - Our method applies to both PTQ and QAT while (Alizadeh et al., 2020) focuses on PTQ.  


- **Quantization methods**
    - As a rule, these works can be classified into two types: 
        1. post-training acceleration
        2. training acceleration
    - Both suffer from one fundamental drawback- they are not robust to variations in quantization process or bit-widths other than the one they were trained for. 

## 3. Model and Problem Formulation 
![](/img/2024-01-16-00-58-56.png){: width="80%"}{: .center-img}

### 3.1 Robustness to varying quantization step size
