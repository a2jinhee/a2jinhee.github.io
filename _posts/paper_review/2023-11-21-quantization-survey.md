---
layout: minimal
title: A Survey of Quantization Methods for Efficient Neural Network Inference
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 231121
---

## A Survey of Quantization Methods for Efficient Neural Network Inference

_2022.11.21_  
 <br>
Quantization Survey
**Keywords: #Quantization**

---

## 0. Abstract
- Problem: In what manner should a set of continuous real-valued numbers be distributed over a fixed discrete set of numbers
    1) to minimized the number of bits required  
    2) and also to maximize the accuracy of the attendant computations

## 1. Introduction 
- Achieving efficient, real-time NNs with optimal accuracy 

### 1) Designing efficient NN model architectures 
- Optimizing NN model architecture in terms of its micro-architecture such as kernel types(depth-wise CONV layer), or macro-architecture such as module types(residual, inception) → manual way 
- AutoML and Neural Architecture Search (NAS) → automated way

### 2) Co-designing NN architecture and hardware together 
- Adapt and co-design the NN architecture for a particular target hardware platform
- A dedicated cache hierarchy can execute bandwidth bound operations much more efficiently 

### 3) Pruning
- Neurons with small *saliency*(sensitivity) are removed, resulting in a **sparse computational graph**
- Unstructured pruning
    - Removing neurons with small saliency
    - Pro: Aggressive pruning with little impact on the generalization performance
    - Con: Sparse matrix operations which are hard to accelerate and typically memory-bound
- Structured pruning
    - A group of parameters (e.g. whole CONV filters) removed
    - Pro: Changing the whole input/outupt shapes of layers, thus permitting dense matrix ops. 
    - Con: Aggressive structured pruning → significant accuracy degradation

### 4) Knowledge Distillation
- Training a large model and using it as a teacher to train a more compact model
- Hard to achieve high compression ratio with KD alone. 
- The combination of KD with quantization and pruning has shown great success

### 5) Quantization 
- Quantization in NN training 
- Proven to be difficult to go below half-precision without significant tuning → quantization for inference 

## 2. General History of Quantization 
- Quantization: A method to map from input values in a large(continuous) set to output values in a small (finite) set
- ex) Shannon's lossless coding theory (Variable-rate quantization), Huffman Coding, Pulse Code Modulation
- Quantization in NN
    1. Inference and training of NNs are both computationally intensive
    → Efficient representation of numerical values is important
    2. Current NNs are heavily over-parametrized
    → Ample opportunity for reducing bit precision without impacting accuracy. 
    3. **NNs are very robust to aggressive quantization and extreme discretization**
    → This new DOF comes from the sheer number of parameters involved
    4. The layered structure of NN models offers an additional dimension to explore
    → Different layers have different impact on the loss function, and this motivates a **mixed-precision approach** to quantization 

    ▵ Thus, it is possible to have high error/distance between quantized model and the original model, while still attaining very good generalization performance. 

## 3. Basic Concepts of Quantization 
### 1) Problem Setup and Notations
  - In quantization, the goal is to reduce the precision of both the parameters, as well as the intermediate activation maps to low-precision, with minimal impact on the generalization power/accuracy of the model. 

<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']],
    displayMath: [['$$', '$$'], ['\[', '\]']]
  },
  svg: {
    fontCache: 'global'
  }
};
</script>
<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js">
</script>