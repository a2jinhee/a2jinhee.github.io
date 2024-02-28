---
layout: default
title: >
    Efficient Acceleration of Deep Learning Inference on Resource-Constrained Edge Devices: A Review
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240216
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

**Keywords: #On-deviceAI**

---

## 0. Abstract
**The Problem of DNNs and DL Algorithms**  
DNNs are often computationally expensive, power-hungry, and require large memory to process complex and iterative operations of millions of parameters.  

**Why do we want to use edge devices for DNNs?**   
Processing on edge devices can significantly reduce cloud transmission cost. Training and inference of DL models are typically performed on high-performance computing (HPC) clusters in the cloud. 

**The main challenge for DL inference on the edge**  
Edge devices have limited memory, computing resources, and power-handling capability. 

**Research directions for DL inference on edge devices**   
1) Novel DL architecture and algorithm design
2) Optimization of existing DL methods ✪
3) Development of algorithm-hardware codesign
4) Efficient accelerator design for DL deployment

## 7. Deep Learning Model Compression for Edge Inference 


