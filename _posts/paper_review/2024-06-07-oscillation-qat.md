---
layout: default
title: >
    Overcoming Oscillations in Quantization-Aware Training
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240607
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

 **Keywords: #QAT #weightdistribution**

---

## 0. Abstract
- This paper delves into the phenomenon of **weight oscillations** and show that it can lead to a *significant accuracy degradation* due to *wrongly estimated BN statistics*.
- We propose two novel QAT algorithms to overcome oscillations during training: 
  - Oscillation dampening
  - Iterative weight freezing

## 1. Introduction
- Main Problem: When using STE for QAT, weights seemingly randomly oscillate between adjacent quantization levels leading to detrimental noise during the optimization process. 
- Problem of weight oscillation: They corrupt the estimated inference statistics of the BN layer collected during training, leading to poor validation accuracy.
  - Pronounced in low-bit quantization of efficient networks with depth-wise separable layers.
  - Probable solution: BN statistics re-estimation 
- Our solution 
  - Oscillation dampening 
  - iterative freezing of oscillating weights

## 2. Oscillations in QAT
### 2.1. Quantization-aware training 
- Original weights: commonly referred to as the *latent weights* or *shadow weights*
- STE: We approximate the gradient of the rounding operator as 1 within the quantization limits.
<!-- $$
\frac{\partial \mathcal{L}}{\partial \mathbf{w}}=\frac{\partial \mathcal{L}}{\partial \widehat{\mathbf{w}}} \cdot \mathbf{1}_{n \leq \mathbf{w} / s \leq p}
$$ --> 

<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%5Cfrac%7B%5Cpartial%20%5Cmathcal%7BL%7D%7D%7B%5Cpartial%20%5Cmathbf%7Bw%7D%7D%3D%5Cfrac%7B%5Cpartial%20%5Cmathcal%7BL%7D%7D%7B%5Cpartial%20%5Cwidehat%7B%5Cmathbf%7Bw%7D%7D%7D%20%5Ccdot%20%5Cmathbf%7B1%7D_%7Bn%20%5Cleq%20%5Cmathbf%7Bw%7D%20%2F%20s%20%5Cleq%20p%7D"></div>

### 2.2. Oscillations
- Toy regression example: Looks complicated, but really just the MSE between label and output of quantized model.
<!-- $$
\min _w \mathcal{L}(w)=\mathbb{E}_{\mathbf{x}}\left[\frac{1}{2}\left(\mathbf{x} w_*-\mathbf{x} q(w)\right)^2\right]
$$ --> 

<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%5Cmin%20_w%20%5Cmathcal%7BL%7D(w)%3D%5Cmathbb%7BE%7D_%7B%5Cmathbf%7Bx%7D%7D%5Cleft%5B%5Cfrac%7B1%7D%7B2%7D%5Cleft(%5Cmathbf%7Bx%7D%20w_*-%5Cmathbf%7Bx%7D%20q(w)%5Cright)%5E2%5Cright%5D"></div>

![](/img/2024-06-08-00-21-07.png){: width="85%"}{: .center-img}

- As the latent (shadow) weight $w$ approaches the optimal value $w_\ast$, it starts oscillating around the decision threshold between the quantization levels above $w_\uparrow$ and below $w_\downarrow$ the optimal value, as opposed to converging to the region closer to the optimal value ~~q(w_\ast)~~ $w_\ast$.
  
- **Why does this happen?** When weights are above the threshold, STE pushes latent weight down towards $w_\downarrow$. When weights are below the threshold, STE push latent weight up towards $w_\uparrow$.
  - When input is fixed, quantized weight will induce positive/negative gradients regardless of whether optimal weight is above/below the threshold. Therefore, weights oscillate near the threshold rather than the optimal value. 
  
- Frequency of oscillation: Dictated by the distance of the optimal value from its closest quantization level 
  - this can be explained. 

### 2.3. Oscillations in practice 
- We observe that many of the weights appear to randomly oscillate between two adjacent quantization levels. 
- After the supposed convergence of the network, a large fraction of the latent weights lie right at the decision boundary between grid points.  
  - This reinforces the observation that a significant proportion of weights oscillate and cannot converge. 
![](/img/2024-06-08-00-34-35.png){: width="100%"}{: .center-img}

#### 2.3.1. The effect on BN statistics
- Many weights oscillate between quantized states, even at the supposed convergence of the network → Output statistics of each layer can vary significantly between gradient updates
- BN layers track the running mean/variance, so that it can be used during *inference*
  - These running estimates can be corrupted due to dist. shift by oscillations
- Degradation in accuracy due to shift induced by oscillations is influnced by two factors: 
  1. Lower the bit-width $b$, the distance becomes larger, as it is proportional to $1/2^b$. 
  2. Number of weights per outpu channel. Smaller the number of weights, the larger the contribution of individual weights to the final accumulation. When number is big enough, the effects of oscillations average out due to the law of large number.  
- BN re-estimation: Is this why BN calibration shows way better accuracy than train-from-scratch? 
- KL divergence to quantify the discrepancy between population and estimated statistics
  - KL divergence is much larger for depth-wise separable layers than in point-wise convolutions. 
  
#### 2.3.2. The effect on training
??

## 4. Overcoming oscillations in QAT

### 4.1. Quantifying Oscillations 
- Calculate frequency of oscillations over time using an exponential moving average (EMA)
- For oscillation to occur, it needs to satisfy: 
  1. Integer value of the weight needs to change 
  2. Direction of change needs to be opposite than that of the previous change. 
- We then track the frequency of oscillations over time using EMA: 
<!-- $$
f^t=m \cdot o^t+(1-m) \cdot f^{t-1}
$$ --> 

<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?f%5Et%3Dm%20%5Ccdot%20o%5Et%2B(1-m)%20%5Ccdot%20f%5E%7Bt-1%7D"></div>

### 4.2. Oscillation dampening
- When weights oscillate, they always move around the decision threshold between two quantization bins. 
- This means that oscillating weights are always close to the edge of the quantization bin. 
- In order to dampen the oscillatory behavior, we employ a regularization term that encourages latent weights to be close to the center of the bin rather than its edge. 
- Dampening loss (similar to weight decay):
  <!-- $$
  \mathcal{L}_{\text {dampen }}=\left\|s \cdot \mathbf{w}_{\text {int }}-\operatorname{clip}(\mathbf{w}, s n, s p)\right\|_F^2
  $$ --> 

<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%20%20%5Cmathcal%7BL%7D_%7B%5Ctext%20%7Bdampen%20%7D%7D%3D%5Cleft%5C%7Cs%20%5Ccdot%20%5Cmathbf%7Bw%7D_%7B%5Ctext%20%7Bint%20%7D%7D-%5Coperatorname%7Bclip%7D(%5Cmathbf%7Bw%7D%2C%20s%20n%2C%20s%20p)%5Cright%5C%7C_F%5E2"></div>

- Final training objective is: $\mathcal{L}=\mathcal{L}_\text{task}+\lambda\mathcal{L}_\text{dampen}$