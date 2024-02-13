---
layout: default
title: PCD to Laser Scan
nav_exclude: true
grand_parent: Projects
parent: Hwasung Autonomous Driving Contest
show_toc: true 
---

## PCD to Laser Scan

### Why do we need to convert PCD to Laser Scan? 
Point Cloud Data (PCD) is one of the most widely used 3D data types out there, along with voxels and polygon mesh. However, what is the use of high-quality 3D data if we don't have the right type of algorithm to tailor it to our needs? While there are many algorithms that directly process 3D data, it is still at the expense of huge power consumption and low throughput. It is safe to say that 3D data processing algorithms are not (yet) efficient enough to replace 2D data processing algorithms. Therefore, we convert pcd to a 2D data type- Laser Scan, in order to process the data easily and in real-time. 

### The Steps 
1. **Prepare the pcd preprocess github repo**: We need to preprocess the pcd before the conversion. Preprocessing steps include voxelization downsampling, RANSAC (removing the ground points), and removing noise and outliers. Further information about the github github repo can be found here [<i class="fa fa-paperclip" aria-hidden="true"></i> PCD Preprocessing](#). 
2. **Configure the launch file of the pcd preprocess github repo**: The conversion process consists of two nodes- the preprocess node, and the conversion node. Once the preprocessing of the pcd is finished, we need to publish the data to the conversion node. We do this by configuring the launch file for the preprocess node, following this site [[ROS] PointCloud2 to LaserScan <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://bigbigpark.github.io/ros/pointcloud_to_laser/). 
3. **Clone the `pointcloud_to_laserscan` github repo**: Clone this github repo [pointcloud_to_laserscan <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/ros-perception/pointcloud_to_laserscan). <br>
<i class="fa-solid fa-triangle-exclamation"></i> The most recent version of this repo is suited for *ROS2*. When using *ROS*, this will cause conflict during the `catkin_make` process. Git clone the appropriate version (branch), which for me, was `lunar_devel`. 
4. **Visualize in RVIZ**: Play the rosbag file, launch both the preprocess node and conversion node, and set RVIZ to see all topics that are being published. We can manually edit what are visualized, by going to 'Add → by topic' and choosing the 3D pcd topics or the 2D laser scan topic we wish to see. 

### The Result
