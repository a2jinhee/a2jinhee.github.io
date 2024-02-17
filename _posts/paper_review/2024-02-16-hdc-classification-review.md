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

#### 2. Binding
- Pointwise multiplication = XOR operation
- Aims to form associations between two related hypervectors
- $X = A \oplus B$ becomes otrthogonal to both $A$ and $B$

#### 3. Permutation
- Circular right bit-shift
- $\text{Ham}(\rho (A), A) \sim 0.5$ for ultra-wide hvs

## 3. The HD Classification Methodology