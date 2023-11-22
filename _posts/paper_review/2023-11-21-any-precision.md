---
layout: minimal
title: Any-Precision Deep Neural Networks 
nav_exclude: true
parent: Paper Review
math: mathjax3
---

## Any-Precision Deep Neural Networks 

_2022.11.21_  
 <br>
 Deep dive into quantization
**Keywords: #Quantization**

---

**0. Abstract**
- github code: https://github.com/SHI-Labs/Any-Precision-DNNs
- Proposal: Any-precision deep neural networks which are trained with a new method that allows the learned DNNs to be flexible in numerical precision during **inference**
- How? By <ins>truncating</ins> the least significant bits. 
- Problem: Previous literature presents solutions to **train** models at fixed precision/accuracy trade-off point. $\rightarrow$ But how to produce a model flexible in **runtime precision** is largely unexplored

**1. Introduction**
- Previous literature
    1. Mixed-precision models: models with some layers processing in ultra-low precision and some layers in high precision 
- Main Problem
    - efficiency/accuracy trade-off always exists, so we have to find the right trade-off point, but what if we demand flexibility? 
    - That is, it would be favorable if we can *dynamically change* the efficiency/accuracy trad-off point given diffenert situations, without re-training or re-calibration 
- Contributions 
    - In runtime, we can quantize its layers into various precision layers without fine-tuning or calibration and without any data . Accuracy changes smoothly w.r.t. its precision level without drastic performance degradation 
    - Any-precision DNN trains better low-bit models with KD 


**2. Related Work**



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
