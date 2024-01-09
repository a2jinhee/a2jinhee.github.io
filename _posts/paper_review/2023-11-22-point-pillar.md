---
layout: minimal
title: >
    PointPillars: Fast Encoders for Object Detection from Point Clouds
nav_exclude: true
parent: Paper Review
math: mathjax3
nav_order: 231122
---

## PointPillars: Fast Encoders for Object Detection from Point Clouds

_2022.11.22_  
 <br>
Pointpillars paper review for 3d object detection competition  
**Keywords: #3dobjectdetection #3dmodel #pointcloud**

---

## 0. Abstract
- Proposal: PointPillars, a novel encoder which utilizes PointNets to learn a representation of point clouds organized in vertical columns (pillars). + a lean downstream network to train the encoded features

## 1. Introduction
- Difference between **2d computer vision** and **lidar pcd object detection**
    1. The point cloud is a sparse representation, while image is dense
    2. The point cloud is 3D, while the image is 2D
- Past Literature
    - 3D convolution → projection of pc to image → pc to bird's eye view(BEV)
    - BEV tends to be extremely sparse → VoxelNet, SECOND uses 3D convolution middle layers
- Proposal
    - PointPillars: a method for object detection in 3D that enables end-to-end learning with only 2D convolutional layers

## 2. PointPillars Network 
1. Feature encoder network that converts pc to a sparse pseudo image
2. 2D convolutional backbone to process the pseudo-image into high-level representation
3. A detection head that detects and regresses 3D boxes. 

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