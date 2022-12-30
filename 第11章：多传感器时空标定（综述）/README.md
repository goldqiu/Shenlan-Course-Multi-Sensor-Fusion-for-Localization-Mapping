# 第11章：多传感器时空标定

## 课程内容

代码、PPT、视频见文件夹

### 课程笔记

#### 多传感器标定简介

![pic1](.\PIC\pic1.jpg)

讲解思路
1) 以思路讲解为主，并给出参考文献和开源代码，不做过多细节展开；
2) 对已有方法做汇总分析，以求能在新的任务中掌握标定方案设计思路。

#### 内参标定

##### 雷达内参标定

目的
由于安装原因，线束之间的夹角和设计不一致，会导致测量不准。
2) 方法
多线束打在平面上，利用共面约束，求解夹角误差。

3) 参考

论文： Calibration of a rotating multi-beam Lidar 论文： Improving the Intrinsic Calibration of a Velodyne LiDAR Sensor 论文： 3D LIDAR–camera intrinsic and extrinsic calibration: Identifiability and analytical least-squares-based  initialization

##### IMU内参标定

新的预积分值=老的预积分值+根据bias的变化量算出的新的预积分结果

1) 目的
由于加工原因，产生零偏、标度因数误差、安装误差。
2) 方法
分立级标定：基于转台；
迭代优化标定：不需要转台。
3) 参考

论文：A Robust and Easy to Implement Method for IMU Calibration without External Equipments
代码：https://github.com/Kyle-ak/imu_tk

##### 编码器内参标定

1) 目的
用编码器输出解算车的位移增量和角度增量，需已知轮子半径和两轮轴距。
2) 方法
以车中心雷达/组合导航做观测，以此为真值，反推模型参数。

3) 参考

论文： Simultaneous Calibration of Odometry and Sensor Parameters for Mobile Robots

##### 相机内参标定

1) 目的
相机与真实空间建立关联，需已知其内参。
2) 方法
张正友经典方法

#### 外参标定

##### 雷达和相机外参标定

1) 目的 

解算雷达和相机之间的相对旋转和平移。 

2) 方法 PnP是主流，视觉提取特征点，雷达提取边缘， 建立几何约束。 

3) 参考 

论文： LiDAR-Camera Calibration using 3D-3D Point correspondences 

代码：https://github.com/ankitdhall/lidar_camera_calibration 论文： Automatic Extrinsic Calibration for Lidar-Stereo Vehicle Sensor Setups 

代码： https://github.com/beltransen/velo2cam_calibration

##### 多雷达外参标定

1) 目的
多雷达是常见方案，使用时将点云直接拼接，但前提是已知雷达之间的外参（相对旋转和平移）。

2) 方法 基于特征(共面)建立几何约束，从而优化外参。

3) 参考 

论文： A Novel Dual-Lidar Calibration Algorithm Using Planar Surfaces 

代码： https://github.com/ram-lab/lidar_appearance_calibration

##### 手眼标定

1) 目的 

手眼标定适用于所有无共视，但是能输出位姿的传感器之间标定。包括：

 • 无共视的相机、雷达，或雷达与雷达之间； 

• 相机与IMU，或雷达与IMU之间(前提是IMU要足够好，或直接使用组合导航)

2) 方法

均基于公式 AX = XB
论文： LiDAR and Camera Calibration using Motion Estimated by Sensor Fusion Odometry
代码： https://github.com/ethz-asl/lidar_align

##### 融合中标定

1) 目的
• 脱离标靶，实现在线标定；
• 某些器件无法提供准确位姿(如低精度IMU)，不能手眼标定。

2)方法

在融合模型中，增加外参作为待估参数。

3)参考

众多vio/lio系统，如vins、lio-mapping、M-Loam 等

##### 总结

1) 这些方法中，推荐优先级从高到低为：
a. 基于共视的标定
b. 融合中标定
c. 手眼标定
2) 建议
应在良好环境下标定，尽量避免不分场景的在线标定。良好环境指观测数据优良的场景，例如：
a. GNSS 信号良好；
b. 点云面特征丰富，没有特征退化；
c. 动态物体较少

#### 时间标定

##### 离散时间

![PIC2](.\PIC\PIC2.jpg)

参考文献：Online Temporal Calibration for Monocular Visual-Inertial Systems

![PIC3](.\PIC\PIC3.jpg)

参考文献：Online Temporal Calibration for Camera-IMU Systems: Theory and Algorithms

##### 连续时间

![PIC4](.\PIC\PIC4.jpg)

参考文献：3D Lidar-IMU Calibration based on Upsampled Preintegrated Measurements for Motion Distortion Correction

2) 方法
把输入建立为连续时间函数，从而可以在任意时间求导。
3) 参考
a. kalibr 系列
论文：Continuous-Time Batch Estimation using Temporal Basis Functions
论文： Unified Temporal and Spatial Calibration for Multi-Sensor Systems
论文： Extending kalibr Calibrating the Extrinsics of Multiple IMUs and of Individual Axes
代码：https://github.com/ethz-asl/kalibr
b. 其他
论文： Targetless Calibration of LiDAR-IMU System Based on Continuous-time Batch Estimation
代码：https://github.com/APRIL-ZJU/lidar_IMU_calib

##### 总结

1) 时间差估计，在某些情况下不得已而为之，实际中应尽量创造条件实现硬同步；
2) 不得不估计时，也应尽量在良好环境下估计。