---
layout: default
title: >
    Review and Benchmarking of Precision-Scalable Multiply-Accumulate Unit Architectures for Embedded Neural-Network Processing
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240510
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

 **Keywords: #Precision-Scalable #Hardware**

---

## 0. Abstract
- Deep learning has come with an enormous computational need for billions of MAC operations per inference. 
- This work.. 
  - Reviews the SOTA precision-scalable MAC architectures 
  - Benchmarked in a 28nm commercial CMOS process
  - Circuits are analyzed for each precision (2~8 bits)
  - Highlight the impact of architectures and scalability in terms of energy, throughput, area and bandwidth. 
  - Understand the key trends to reduce computation costs in NN processing

## 1. Introduction
- Main challenge of **'edge intelligence'**: Limited energy budget of embedded devices ↔ Computationally-intensive deep-learning algorithms, requiring billions of MAC operations. 
  - Algorithmic level
  - System-architecture level 
  - Cricuit level 

- **Reduced-precision computing**: Demonstrates large benefits with low or negligible impact on network accuracy. → Shown that the optimal precision can vary within a neural network itself (layer to layer, channel to channel)

- Problem (?? what does this mean)
1. Many run-time precision-scalabe MAC architectures have been implemented in various process technologies, with nonidentical bitwidths or scalability levels, or have been integrated into entirely different systems. 
2. Few works evaluate their performances against a baseline design (i.e. a MAC unit w/o scalability) to clearly show the configurability overheads. 
3. Many designs are based on ad-hoc scalability techniques, lacking a systematic approach and a clear analysis of which scalability principles are common,respectively distinct, between different implementations.

- Contributions: 
    - Section II: 
    - Section III: 
    - Section IV-V: 
    - Section VI: 
    - Section VII: 

## 2. Dataflow Implications of Precision Scalability
