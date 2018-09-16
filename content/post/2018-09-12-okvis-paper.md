---
title: "OKVIS要点速览"
tags: ["VIO", "SLAM" ]
date: "2018-09-12"
---

主要是策略

<!--more-->

## 前端
### 1.特征点匹配
特征点 ： a customized multi-scale SSE-optimized Harris corner detector + BRISK descriptor

匹配： 使用imu获取一个状态的先验估计.
一个local map有 3d位置确定的大量 landmarks. 对这些点：  
1. 对能投到当前帧的点 3D-2D BF-matching  
2. outlier rejection: 1. 使用先验的不确定估计（by imu）， 进行 Mahalanobis test. 2. RANSAC

对于没有3d位置的，2d-2d匹配:
1. 首先BF-matching， 然后进行三角化
2. 只有深度不确定性比较小的点才认为初始化成功，否则放到后续帧的matching中
3. 进行当前帧和newest keyframe的 RANSAC

### 2.关键帧选取
在一下情况添加关键帧：  
1. 能够投影到/匹配上 当前帧的点的数量小于 50%  
2. 匹配和提取的点数比值小于 20%

## 系统的边缘化
系统维护一个sliding window，当新增一个keyframe时，会marginalize window中比较老的一帧以及相关的landmark.  
为了确保矩阵的稀疏性（对于公视点，marginalize掉这些点，会增加帧与帧之间的link， 其稀疏性会被破坏； 即要marg不被其他keyframe观测到的特征点)  

另外，系统采用FEJ，其线性化点需要固定下来。  
系统进行marg的时候会用到jacobian，一般来说，我们会用最新的估计值再计算一个jacobian；  
但是如果每次都使用最新的线性化点去计算，其结果导致了 yaw的能观性被改变，而这是不符合系统的固有特性的；  
故huang 提出了FEJ， 在marg的时候都使用同一个线性化点， 保证系统的特性不被人为改变。

最后吐嘈一下，OKVIS的代码，写的真的是... 蒙逼好吧  
太太太...模块化了~  
虽然很牛逼，但是读起来一层套一层就比较吃力。  
还是VINS清纯不做作，顺着走下来，很舒服~