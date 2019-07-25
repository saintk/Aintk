---
title: "Cartographer 分析"
tags: ["SLAM" ]
date: "2019-04-12"
---

学习一下Cartographer。

<!--more-->

## Overview
![overview](/media/posts/2019-04-12-cartographer-code-reading/high_level_system_overview.png)


主要涉及三篇文章
    1.1  Real-Time Loop Closure in 2D LIDAR SLAM , ICRA 2016
    1.2  Efficient Sparse Pose Adjustment for 2D Mapping (SPA)
    1.3  Real-Time Correlative Scan Matching (BBS)

1) Real-Time Loop Closure in 2D LIDAR SLAM
文章的重点是第四部分和第五部分
  第四部分：local 2d slam, 主要是将 局部地图的 scan matching作为一个二次型优化问题由ceres slover 解决      
  第五部分： closing loop， 采用了 SPA（Sparse Pose Adjustment）进行后端loop closure。 这个过程中有一个很重要的过程是的scan和 submap的匹配，这（Branch-and-bound scan matching）, 它可大幅提高精度和速度.
  cartographer的整体架构是典型的 前端建图 （局部地图）+后端优化。 整个性能相比传统的 state-of-the-art的算法没有什么太大的提高， 而是更像一个实用性的产品，很好的平衡了性能和速度。

## Closing Loops
相对Pose存储在memory用于loop closing optimzation. 一旦子图固定，当其他的pairs检测到相对于这部分的scan，就发生了回环检测。

A.优化问题
回环检测的优化问题，可以建模为Sparse Pose Adjustment.

---
待续