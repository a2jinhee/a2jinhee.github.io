---
layout: default
title: >
    Improving Low-Precision Network Quantization via Bin Regularization
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240503
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

 **Keywords: #Quantization #WeightDistribution**

---

## 0. Abstract
- We propose a novel weight regularization algorithm 
- Instead of constraining the overall data distribution, we optimize all elements in each quantization bin to be as close to the target quantized value as possible. 
- Such bin regularization (BR) mechanism encourages the weight distribution of each quantization bin to be sharp/approximate to a Dirac delta distribution ideally. 

## 1. Introduction
- Usually, the network accuracy decreases and the hardware performance increases as bit precision decreases. → PTQ, QAT
- We propose a novel regularization algorithm for improving low-precision network quantization. 
  - We hypothesize that quantization error will approach zero if all FP elements in each bin are close enough to the target quantized value. 
  - Our bin regularization is expected to reduce the quantization error at the bin level
  - Our bin regularization also encourages sharp distribution for each quantization bin. 

## 3. Methodology
### 3.1. Quantization Baseline
