---
title: "使用ArUco定位"
tags: ["VIO" ]
date: "2018-06-10"
---

这篇文章介绍了如何使用ArUco库进行定位。
<!--more-->

## 1. 配置安装

ArUco库下载:   
https://sourceforge.net/projects/aruco/files/  
这里我使用的是3.0.10版本,按照readme安装。(make install) 

编译安装MarkerMapper:   
https://sourceforge.net/p/markermapper/activity/?page=0&limit=100#5addf1323241d20a481f2a02  
这里我使用的是1.0.12版本，按照readme安装。(make install) 

## 2. 采集数据并构图
参照Marker_Mapper中的Readme文件，执行一下命令:
```cpp
./utils/mapper_from_video video.avi camera.yml 0.1 -out markerset -d dict
```
其中为拍摄有marker的视频，camera.yml为相机内参，0.1为相关的tag大小，
video.avi:拍摄有marker的视频.  
camera.yml:相机内参.  
0.1：为marker的大小，按实际情况给定.  
-o map: 生成的文件.
-d <dict>: 制定使用的marker字典，按照所使用的marker指定，有ARUCO_MIP_16h3 ARUCO_MIP_25h7 ARUCO_MIP_36h12等.  
![ArUco采集视频](/media/posts/ArUco/arUco_marker_1.png)
如图所示: 我们所用的dict为ARUCO_MIP_36h12
![map_set](/media/posts/ArUco/map_viewer_screenshot_10.06.2018.png)
最后我们将得到markerset.yml以及相应的pcd文件,如果结果markerset的结果不好请检查相机标定参数和marker的参数.

## 3. 使用api获取当前位置
通过调用aruco的相关接口，来获取当前位置.
有四个需要定义的接口:  

* CameraParameters TheCameraParameters;     // 相机内参
* MarkerMap TheMarkerMapConfig;             // 地图参数
* MarkerDetector TheMarkerDetector;         // Marker检测
* MarkerMapPoseTracker TheMSPoseTracker;    // 位置跟踪接口

具体可以使用也很简单，可以参照官网的接口.   
对每一帧图像可以通过如下操作获得相机位姿.
```cpp
    //读取marker_map和相机参数
    TheMarkerMapConfig.readFromFile(inputArucoMapPath); 
    TheCameraParameters.readFromXMLFile(cameraParamPath); 
    //获取建图所使用的字典
    TheMarkerDetector.setDictionary(TheMarkerMapConfig.getDictionary()); 
    //获取marker的size
    if (TheMarkerMapConfig.isExpressedInPixels() && TheMarkerSize > 0)
        TheMarkerMapConfig = TheMarkerMapConfig.convertToMeters(TheMarkerSize);
    if (TheCameraParameters.isValid() && TheMarkerMapConfig.isExpressedInMeters()) {
        TheMSPoseTracker.setParams(TheCameraParameters, TheMarkerMapConfig);
        TheMarkerSize = cv::norm(TheMarkerMapConfig[0][0] - TheMarkerMapConfig[0][1]);
    }

    //开始跟踪
    vector<aruco::Marker> detectedMarkers = TheMarkerDetector.detect(TheInputImage);
    if (TheMSPoseTracker.isValid()) {
        if (TheMSPoseTracker.estimatePose(detectedMarkers)) 
            //获得相机当前的位姿
            cv::Mat T = TheMSPoseTracker.getRTMatrix(); 
    }
```
实测精度可以达到厘米级。
