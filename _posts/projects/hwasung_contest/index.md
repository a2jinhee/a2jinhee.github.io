---
layout: default
title: Hwasung Autonomous Driving Contest
parent: Projects
has_children: true
has_toc: false
nav_order: 0
---

## Hwasung Autonomous Driving Contest

_2023.10.13~2023.10.14_  
<br>
I participated in the *Localization Team*, *Vision Team*, and the *Path-Planning Team*. 

---

### Local Team

- Processing LiDAR Data
  - 3D lidar points (pcd) → 2D laser scan: [pcd-to-laserscan(doc)](https://www.notion.so/pcd-to-laserscan-a65a06f5dbe3456f82467b6c2707aaef?pvs=21)
  - pcd(point cloud data) processing ex) clustering
- Detect parking space using lidar data
  - algorithm
  - clustering
- Setting up the IMU, GPS driver

<br>

### Vision Team

- YOLOP: Lane Detection
  - Post-processing algorithm
  - TensorRT engine
- YOLOv5: traffic light, object detection
  - Collecting dataset
  - Lidar-Camera sensor fusion
- PersFormer: 3D Lane Detection
  - CMake
  - nms
- CUDA, torch, nvidia driver compatibility
- Lane detection algorithm

<br>

### Path-Planning Team

- Morai Simulator
  - Robot Localization package
  - map, odom, base frame tf
  - navigation stack
  - writing roslaunch files
- HD Map
  - QGIS: editing maps
  - UTM, WGS84
- ROS Navigation Stack
  - ROS service
  - Goal manager logic
