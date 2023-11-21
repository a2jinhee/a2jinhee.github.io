---
layout: minimal
title: >
  Neural Architecture Search: A Survey
nav_exclude: true
parent: Paper Review
math: mathjax3
---

## Neural Architecture Search: A Survey

_2022.11.12_  
 <br>
Greath introduction to Auto ML 
**Keywords: #AutoML #NAS**

---

### 1. Introduction

- The success of deep learning is due to its automation of the **feature engineering process**. 
- Neural Architecture Search (NAS) is the process of automating **architecture engineering**.   

<br>
```mermaid
flowchart LR;
	1["Search Space"] --> 287661["Search Strategy"]
	287661 -->|"architecture A"| 340211["Performance\nEstimation Strategy"]
	340211 -->|"performance estimate of A"| 287661
```
<br>

- **Search Space**
  - Defines which architectures can be represented.
  - Incorporating prior knowledge can reduce searh sapce
  - This introduces **human bias**, which prevents finding novel architectures beyond human knowledge
  - (ex) CONV 3x3, avgpool, max
- **Search Stratergy**
  - Defines how to explore the search space
  - Exploration-exploitation trade-off: If the search process only exploits known configurations, it may miss out on potentially superior designs. On the other hand, if it explores too much, it might waste resources testing suboptimal options.
  - (ex) grid search, random search, evolutionary
- **Performance Estimation Strategy**
  - Objective is to find architectures that achieve high predictive performance on unseen data. 
  - Recent research focus on reducing the cost of performance estimations. 


### 2. Search Space
- **chain-structured neural networks**
  1. the (max) number of layers n  
  2. the type of ops. every layer executes (ex) pooling, conv.. 
  3. hyperparameteers associated with the ops. (ex) filter #, kernel size, strides for conv 
- **multi-branch networks** 
  - The input of a certain layer can be a function combining previous layer outputs $\rightarrow$ more degrees of freedom
  1. chain-structured networks by setting input as a function combining previous layers
  2. Residual Networks: previous layer outputs are summed
  3. DenseNets: previous layer outputs are concatenated

- **Using "repeated motifs"**
  - search for *cells or blocks* instead of whole architectures 
  1. *normal cell*: preserves the dim. of the input
  2. *reduction cell*: reduces the spatial dim. 
  - The final architecture is built by stacking these cells in a predefined manner

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
