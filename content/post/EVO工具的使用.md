---
title: "使用evo评估VINS-MONO"
tags: ["VIO" ]
date: "2018-06-07"
---

EVO工具用于评估SLAM算法在现有数据集上的效果。  
源码在 https://github.com/MichaelGrupp/evo  
目前支持 TUM KITTI Euroc 等格式

evo工具提供了3种误差评估方式:

* `evo_ape` -absoulte pose error  
* `evo_rpe` -relative pose error  
* `evo_rpe` -for-each -sub-sequence-wise averaged pose error

4种工具：

* `evo_traj` - tool for analyzing, plotting or exporting one or more trajectories
* `evo_res` - tool for comparing one or multiple result files from evo_ape or evo_rpe
* `evo_fig` - (experimental) tool for re-opening serialized plots (saved with --serialize_plot)
* `evo_config` - tool for global settings and config file manipulation

## 1. euroc数据集
以VINS为例子介绍如何使用evo评估其在 euroc数据集上的效果。
数据集采用 V1_02_medium

euroc的评估支持 .csv的groudtruth 和 tum 格式的轨迹文件.
首先修改vins源码.
打开 visulization.cpp, 找到 写文件部分代码，并修改为:

```c++
        // write result with tum format, for evo tools
        ofstream foutC(VINS_RESULT_PATH, ios::app);
        foutC.setf(ios::fixed, ios::floatfield);
        foutC.precision(10);
        foutC << header.stamp.toSec()<< " ";
        foutC.precision(5);
        foutC << estimator.Ps[WINDOW_SIZE].x() << " "
              << estimator.Ps[WINDOW_SIZE].y() << " "
              << estimator.Ps[WINDOW_SIZE].z() << " "
              << tmp_Q.w() << " "
              << tmp_Q.x() << " "
              << tmp_Q.y() << " "
              << tmp_Q.z() << endl;
        foutC.close();
```

现在我们有了 tum格式的轨迹输出，以及数据集提供的真值state_groundtruth_estimate0/data.csv  
准备工作已经完成，可以使用evo工具进行评估了 

首先可们可以使用 evo_traj将轨迹画出来.
```cpp
evo_traj tum vins_result_no_loop.txt -p --plot_mode=xyz
```
![vins轨迹](/media/posts/EVO工具的使用/vins_result_no_loop.png)

使用evo_ape确定轨迹误差：
```cpp
evo_ape euroc data.csv vins_result_no_loop.txt -va --plot --save_results result_vins.zip
```
结果如下：
![vins_ape轨迹](/media/posts/EVO工具的使用/ape_map.png)
![vins_ape_raw](/media/posts/EVO工具的使用/ape_raw.png)

这个工具非常好的一点就是，不用去管坐标系的相对变化。两个体坐标系之间的变换，两个世界系的变换，都不用考虑，它会自动调整，并将相对变换输出出来.

## 2.使用自己的数据集
更多情况下，我们需要自己采集数据并评估。  
通过采集vicon数据，我们可以制作自己的真值，直接做成tum的真值格式，方便使用和计算。  
需要注意的是，各传感器之间的时间同步。

首先可以画出轨迹：
```cpp
evo_traj tum vins_tum_result.txt -p --plot_mode=xyz
```
![vins_tango_traj](/media/posts/EVO工具的使用/tango_traj.png)
![vins_tango_traj_xyz](/media/posts/EVO工具的使用/tango_traj_xyz.png)
![vins_tango_traj_rpy](/media/posts/EVO工具的使用/tango_traj_rpy.png)

APE：  
```cpp
evo_ape tum vicon_gt.txt vins_tum_result.txt -va --plot --save_results result_vins.zip
```
![vins_tango_ape_map](/media/posts/EVO工具的使用/tango_ape_map.png)
![vins_tango_ape_raw](/media/posts/EVO工具的使用/tango_ape_raw.png)

最终的结果会保存在 result_vins.zip中  
`{"std": 0.05500716121872294, "rmse": 0.14635270998813427, "sse": 10.645300513272854, "max": 0.3279892445412863, "min": 0.02701627306479177, "median": 0.12761697807728675, "mean": 0.13562200387668794}`

## 3.小结
总的来说，这个工具是用起来还是非常方便的，很棒！