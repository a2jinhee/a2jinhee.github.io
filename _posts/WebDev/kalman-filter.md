---
layout: default
title: Kalman Filter
parent: Robotics
nav_order: 1
---

<style>
  .container{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
  }
</style>

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

# Kalman Filter

---

_Reference: [선형 칼만 필터의 원리 이해](https://gaussian37.github.io/ad-ose-lkf_basic/)_
<br>

> ## 🍇 Why do we need a Kalman Filter?

- To estimate the position of a robot precisely.
- The state of a robot is portrayed as `(position, velocity)`
- Position and velocity both have some sort of uncertainty, which can be interpreted to some form of a Gaussian distribution.

<br>

> ## 🍇 Correlation between position and velocity

- In real life, position and velocity has some sort of correlation.  
  _ex) lower velocity leads to shorter distances, and higher velocity leads to further distances_
- The goal for Kalman filter  
  → Finding the correlation between position and velocity  
  → This correlation is called a **'covariance matrix'**

| ![](https://gaussian37.github.io/assets/img/autodrive/ose/lkf_basic/01.png) | ![](https://gaussian37.github.io/assets/img/autodrive/ose/lkf_basic/02.png) |
| :-------------------------------------------------------------------------: | :-------------------------------------------------------------------------: |
|                                Uncorrelated                                 |                                 Correlated                                  |

<br>

> ## 🍇 How Kalman Filter Works - the math

> ### 🏳️ Finding the prediction matrix , $$F_k$$

<br>
<div class="container">
<img src="https://gaussian37.github.io/assets/img/autodrive/ose/lkf_basic/05.jpg">
</div>

- Let $$\hat x $$ be the state function.

  $$
  \hat x_k = \begin{bmatrix}
  p_k\\
  v_k
  \end{bmatrix}
  $$

- Transformation Matrix : the matrix that uses the past state $$(p_{k-1}, v_{k-1})$$ to estimate the current state $$(p_{k}, v_{k})$$

  $$
  \hat x_k = F_k \hat x_{k-1}
  $$

- If all is true, the following can be said.

$$
\begin{matrix}
p_k = p_{k-1}+\Delta t v_{k-1}\\
v_k = v_{k-1}
\end{matrix}
$$

$$
F_k = \begin{bmatrix}
1 & \Delta t\\
0 & 1
\end{bmatrix}
$$

<br>

> ### 🏳️ Covariance Scaling

- Covariance Scaling ([proof](https://dlsun.github.io/probability/cov-properties.html))

  $$
  Cov(Ax)=ACov(x)A^T
  $$

<br>

> ### 🏳️ Finding $$p-v$$ Covariance

- Covariance between position and velocitiy is like the below:

  $$
  P_k = \begin{bmatrix}
  pp & pv\\
  vp & vv
  \end{bmatrix}
  $$

- Now, with the prediction matrix and covariance scaling we can derive the formula.

  $$
  \hat x_k = F_k \hat x_{k-1}
  $$

  $$
  Cov(Ax)=ACov(x)A^T
  $$

  $$
  P_k = F_k P_{k-1}F_k^T
  $$

- proof
<br>
<div class="container">
  <img src="https://i.pinimg.com/564x/57/a1/a8/57a1a8bc70b56c22353e7813f09a89a1.jpg" width="60%">
</div>

<br>

> ## 🍇 External Factors
