---
layout: minimal
title: Any-Precision Deep Neural Networks 
nav_exclude: true
parent: Paper Review
---

## Any-Precision Deep Neural Networks 

_2022.11.21_  
 <br>
 Deep dive into quantization
**Keywords: #Quantization**

---

## 0. Abstract
- github code: https://github.com/SHI-Labs/Any-Precision-DNNs
- Proposal: Any-precision deep neural networks which are trained with a new method that allows the learned DNNs to be flexible in numerical precision during **inference**
- How? By <ins>truncating</ins> the least significant bits. 
- Problem: Previous literature presents solutions to **train** models at fixed precision/accuracy trade-off point. $\rightarrow$ But how to produce a model flexible in **runtime precision** is largely unexplored

## 1. Introduction
- Previous literature
    1. Mixed-precision models: models with some layers processing in ultra-low precision and some layers in high precision 
- Main Problem
    - efficiency/accuracy trade-off always exists, so we have to find the right trade-off point, but what if we demand flexibility? 
    - That is, it would be favorable if we can *dynamically change* the efficiency/accuracy trad-off point given diffenert situations, without re-training or re-calibration 
- Contributions 
    - In runtime, we can quantize its layers into various precision layers without fine-tuning or calibration and without any data . Accuracy changes smoothly w.r.t. its precision level without drastic performance degradation 
    - Any-precision DNN trains better low-bit models with KD 


## 2. Related Work
- Low-Precision Deep Neural Networks
    - Binarized Neural Networks, XNOR-Net 
    - Quantization Operator
    - Straight Through Estimator(STE)

## 3. Any-Precision Deep Neural Networks
### 3.1 Overview  

<!-- $$
y = w \cdot x + b
$$ --> 
<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?y%20%3D%20w%20%5Ccdot%20x%20%2B%20b"></div>
- N-bit fixed-point integers to represent weights($w_Q$) and input activations($x_Q$)
<!-- $$
y' = s*(w_Q \cdot x_Q) + b
$$ --> 
<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?y'%20%3D%20s*(w_Q%20%5Ccdot%20x_Q)%20%2B%20b"></div>
- Adding a layer-wise scaling factor $s$ helps reduce the output range variation and hence achieve better model accuracy. 
- The activations $y'$ are then quantized into N-bit fixed-point integers

### 3.2 Inference
- **Once training is finished,** we can keep the weights at a higher precision level for storage e.g. 8 bits
- We quantize the weights into lower bit-width by bit-shifting (**truncate**)

### 3.3 Training 

#### 3.3.1 Quantization-aware training (QAT)
**"Train the model taking quantization into consideration"**
- A full precision copy of the weights W is maintained throught the training. 
- Small gradients are accumulated without loss of precision → gradients effect the 'full precision copy of weights' 
- Once the model is trained, only the quantized weights are used for inference. 
- **BN layer is important in low-precision DNN** 
- $y'$ is first passed into the BN layer and THEN quantized into $y_Q$
    
#### 3.3.2 Weights 

- **Uniform quantization strategy**: 
    - Normalize weights to [0,1]:  
    [graph](https://www.desmos.com/calculator/1du8fwsskp)  
    <!-- $$
    w' = \frac{tanh(w)}{2\max(|tanh(w)|)}+0.5
    $$ --> 
    <div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%20%20%20%20%20%20%20%20w'%20%3D%20%5Cfrac%7Btanh(w)%7D%7B2%5Cmax(%7Ctanh(w)%7C)%7D%2B0.5"></div>   
    - Quantize normalized value into N-bit integer     

    <!-- $$
    w_Q' = INT(round(w'* MAX_N)) \quad s' = 1/MAX_N
    $$ --> 
    <div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%20%20%20%20%20%20%20%20w_Q'%20%3D%20INT(round(w'*%20MAX_N))%20%5Cquad%20s'%20%3D%201%2FMAX_N"></div> 
    where $MAX_N$ denotes the upper-bound of N-bit integer
    - Values remapped to approximate the range of floating point values to obtain  

    <!-- $$
    w_Q = 2*w_Q'-1 \quad s = \mathbb{E}(|w|)/MAX_N
    $$ --> 
    <div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%20%20%20%20%20%20%20%20w_Q%20%3D%202*w_Q'-1%20%5Cquad%20s%20%3D%20%5Cmathbb%7BE%7D(%7Cw%7C)%2FMAX_N"></div> 
    - Approximate $w$ with $s*w_Q$

- Forward Pass
    - Execute feed-forward pass with quantized weights:  

    <!-- $$
    y' = s*(w_Q \cdot x_Q) + b
    $$ --> 
    <div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%20%20%20%20%20%20%20%20y'%20%3D%20s*(w_Q%20%5Ccdot%20x_Q)%20%2B%20b"></div>
    Overload of notation 's' - (1) quantization scaling factor for weights, (2) layer-wise scaling factor to reduce output range variation 

- Backward Pass
    - Gradients are computed w.r.t the underlying float-value variable $w$(because we use the quantized version $w_Q$, in the feed-forward pass) and updates are applied to $w$ as well.
    - **Straight through estimator (STE)**: Used to approximate gradients   
        (ex) round op. has zero derivatives everywhere (why? because it's a step function). With STE, the gradient of round op. is 1. 

#### 3.3.3 Activations 
- Clipping: Clip acitvation values to be within [0,1]

#### 3.3.4 Dynamic Model-wise Quantization 
- Previous works: Bit-width N is fixed during the training process → In runtime if we alter N, accuracy drops drastically 
- **Proposal: Dynamically change N within the training stage to align the training and inference process**
- Problem 1: Distribution of activations varies under different bit-width N, especially when N is small, failing to converge 
- Solution 1: Adopt BN layer that calculates running average
- Problem 2: Still fails
- Solution 2: (Proposal) Adopt **dynamically changed BN layer** to work with different N in training. Parameters of all BatchNorm layers are kept after training and used in inference. Storage for each bitwidth BN layer is negligible. 
![](/img/2023-11-25-01-31-57.png){: width="50%"}{: .center-img}

#### 3.3.5 Etc. 
- Knowledge distillation: KD gives ~$\pm$1% boost

## 4. Experiments
### 4.1 Implementation Details
- For all models, first and last layer is real-valued
