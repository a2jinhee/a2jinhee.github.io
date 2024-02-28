---
layout: default
title: >
    Classification using Hyperdimensional Computing: A Review
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 240216
---

## {{page.title}}
*{{ page.url | split: '/' | last | slice: 0, 10}}*

 <br>

**Keywords: #HDC**

---

## 0. Abstract
- ***hypervectors***: Unique data type for HD computing 

## 1. Introduction
- HD computing represents different types of data using *hypervectors*, whose dimensionality is in the thousands (e.g. 10,000-$d$)
- Hypervectors are composed of IID components (binary, integer, real or complex)
- ✪ The concept of ***orthogonality***

## 2. Background on HD Computing 

### A. Classical Computing vs HD Computing 
- Three key factors of computing: 1) Data representation, 2) data transformation, and 3) data retrieval
    - Data representation: Hypervector
    - Data transformation: Add-Multiply-Permute
    - Data retrieval: ?
- *class* hypervector: hvs generated from training data
- *query* hypervector: hvs generated from the test data 
![](/img/2024-02-16-22-19-16.png){: width="90%"}{: .center-img}

### B. Data Representation
- Divided into two categories: binary or non-binary (bipolar, integer)
    - Non-binary hv: More hardware-friendly
    - Binary hv: Higher accuracy

### C. Similarity Measurement
- Non-binary hv: *cosine similarity*
    - Only depend on *orientation*, focusing on the angle and ignoring the impact of magnitude. 
- Binary hv: *normalized Hamming distance*
- Orthogonality in high dimensions: Randomly generated hypervectors are nearly orthogonal to each other when $d$ is high. 
- Orthogonal hvs = dissimilar: Operations  performed on theses orthogonal hvs can form associations or relations.  

![](/img/2024-02-16-23-01-26.png){: width="50%"}{: .center-img}

### D. Data Transformation

#### 1. Bundling 
- Pointwise addition
- Majority Rule

#### 2. Binding
- Pointwise multiplication = XOR operation
- Aims to form associations between two related hypervectors
- $X = A \oplus B$ becomes otrthogonal to both $A$ and $B$

#### 3. Permutation
- Circular right bit-shift
- $\text{Ham}(\rho (A), A) \sim 0.5$ for ultra-wide hvs

## 3. The HD Classification Methodology
### A. The HD Classification Methodology 
1. The encoder employs randomly generated hvs to map training data to HD space. 
2. A total of $k$ class hypervectors are trained and stored in the associative memory. 
3. Inference phase: the encoder generates the query hv for each test data
4. Similarity check between the query hv and class hv in the *associative memory*. 
5. The label with the closest distance is returned. 
![](/img/2024-02-28-20-41-35.png){: width="60%"}{: .center-img}

### B. Encoding Methods for HD Computing 
- Encoding: Mapping input data into hvs, somewhat similar to *extraction of features*
- **Record-based Encoding**  
    - Position hvs: Randomly generated to encode the feature position information → Orthogonal to each other  
    - Level hvs: The feature value information is quantized to $m$ level hvs. For an $N$-dim feature, a total of $N$ level hvs should be generated, which are chosen from the $m$ level hvs. → Randomly bit-flip $d/m$ ($d$ is dimension) to generate next level hv, the last-level hv being nearly orthogonal to the first hv. 
    - Encoding: Bind each position hv with its level hv. The final encoding hv $\mathbf{H}$ is obtained by bundling. 
<!-- $$
\begin{aligned}
& \mathbf{H}=\overline{\mathbf{L}}_1 \oplus \mathrm{ID}_1+\overline{\mathbf{L}}_2 \oplus \mathrm{ID}_2+\cdots+\overline{\mathbf{L}}_N \oplus \mathrm{ID}_N \\
& \overline{\mathbf{L}}_i \in\left\{\mathbf{L}_1, \mathbf{L}_2, \cdots, \mathbf{L}_m\right\}, \text { where } 1 \leq i \leq N
\end{aligned}
$$ --> 

<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%5Cbegin%7Baligned%7D%0A%26%20%5Cmathbf%7BH%7D%3D%5Coverline%7B%5Cmathbf%7BL%7D%7D_1%20%5Coplus%20%5Cmathrm%7BID%7D_1%2B%5Coverline%7B%5Cmathbf%7BL%7D%7D_2%20%5Coplus%20%5Cmathrm%7BID%7D_2%2B%5Ccdots%2B%5Coverline%7B%5Cmathbf%7BL%7D%7D_N%20%5Coplus%20%5Cmathrm%7BID%7D_N%20%5C%5C%0A%26%20%5Coverline%7B%5Cmathbf%7BL%7D%7D_i%20%5Cin%5Cleft%5C%7B%5Cmathbf%7BL%7D_1%2C%20%5Cmathbf%7BL%7D_2%2C%20%5Ccdots%2C%20%5Cmathbf%7BL%7D_m%5Cright%5C%7D%2C%20%5Ctext%20%7B%20where%20%7D%201%20%5Cleq%20i%20%5Cleq%20N%0A%5Cend%7Baligned%7D"></div>  

- **N-gram-based encoding**
    - Random level hvs are generated, then feature values are obtained by permuting these level hvs. 
    - (ex) the level hv $\bar{\mathbf{L}_i}$ corresponding to the $i$-th feature position is rotationally permuted by $(i-1)$ positions. 
    - The final encoded hypervector $\mathbf{H}$ is obtained by binding each *permuted level hvs*.
<!-- $$
\begin{aligned}
& \mathbf{H}=\overline{\mathbf{L}}_1 \oplus \rho \overline{\mathbf{L}}_2 \oplus \cdots \oplus \rho^{N-1} \overline{\mathbf{L}}_N \\
& \overline{\mathbf{L}}_i \in\left\{\mathbf{L}_1, \mathbf{L}_2, \cdots, \mathbf{L}_m\right\}, \text { where } 1 \leq i \leq N
\end{aligned}
$$ --> 

<div align="center"><img style="background: white;" src="https://latex.codecogs.com/svg.latex?%5Cbegin%7Baligned%7D%0A%26%20%5Cmathbf%7BH%7D%3D%5Coverline%7B%5Cmathbf%7BL%7D%7D_1%20%5Coplus%20%5Crho%20%5Coverline%7B%5Cmathbf%7BL%7D%7D_2%20%5Coplus%20%5Ccdots%20%5Coplus%20%5Crho%5E%7BN-1%7D%20%5Coverline%7B%5Cmathbf%7BL%7D%7D_N%20%5C%5C%0A%26%20%5Coverline%7B%5Cmathbf%7BL%7D%7D_i%20%5Cin%5Cleft%5C%7B%5Cmathbf%7BL%7D_1%2C%20%5Cmathbf%7BL%7D_2%2C%20%5Ccdots%2C%20%5Cmathbf%7BL%7D_m%5Cright%5C%7D%2C%20%5Ctext%20%7B%20where%20%7D%201%20%5Cleq%20i%20%5Cleq%20N%0A%5Cend%7Baligned%7D"></div> 

- **Kernel encoding** ✪

### C. Benchmarking Metrics in HD Computing 
- Tradeoff between accuracy and efficiency. 
- **Accuracy**: Appropriate choice of encoding method, and retraining iteratively (compared to single-pass training) improves the training accuracy. 
- **Efficiency**
    - Algorithm: Dimension reduction
    - Hardware: Binarization (employing binary hvs instead of non-binary model), Quantization, Sparsity → HW acceleration anticipated combining HD computing with in-memory computing 