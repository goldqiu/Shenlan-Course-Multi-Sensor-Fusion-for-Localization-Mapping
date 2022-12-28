# 第10章：基于优化的定位方法

## 课程内容

代码、PPT、视频见文件夹

### 课程笔记

#### 基于图优化的定位简介

##### 核心思路

![pic1](.\PIC\pic1.jpg)

![PIC2](.\PIC\PIC2.jpg)

##### 定位流程

整个流程：不断往滑窗里添加新信息，并边缘化旧信息。
需要注意的是：
1) 正常行驶时，不必像建图那样，提取稀疏的关键帧；
2) 停车时，需要按一定策略提取关键帧，但删除的是次新帧，因此不需要边缘化。

![PIC3](.\PIC\PIC3.jpg)

#### 边缘化原理及应用

##### 边缘化原理

![pic4](.\PIC\pic4.jpg)

![pic5](.\PIC\pic5.jpg)

##### 从滤波角度理解边缘化

![pic6](.\PIC\pic6.jpg)

![PIC7](.\PIC\PIC7.jpg)

![PIC8](.\PIC\PIC8.jpg)

新的预积分值=老的预积分值+根据bias的变化量算出的新的预积分结果

![pic9](.\PIC\pic9.jpg)

![PIC10](.\PIC\PIC10.jpg)

![PIC11](.\PIC\PIC11.jpg)

![PIC12](.\PIC\PIC12.jpg)

![PIC13](.\PIC\PIC13.jpg)

#### 基于kitti的实现原理

##### 基于地图定位的滑动窗口模型

![PIC14](.\PIC\PIC14.jpg)

![PIC15](.\PIC\PIC15.jpg)

![PIC16](.\PIC\PIC16.jpg)

![PIC17](.\PIC\PIC17.jpg)

![PIC18](.\PIC\PIC18.jpg)

![PIC19](.\PIC\PIC19.jpg)

##### 边缘化过程

![PIC20](.\PIC\PIC20.jpg)

![PIC21](.\PIC\PIC21.jpg)

![PIC22](.\PIC\PIC22.jpg)

![PIC23](.\PIC\PIC23.jpg)

![PIC24](.\PIC\PIC24.jpg)

![PIC25](.\PIC\PIC25.jpg)

#### lio-mapping介绍

##### 核心思想

基于滑动窗口方法，把雷达线/面特征、IMU预积分等的约束放在一起进行优化。

![PIC26](.\PIC\PIC26.jpg)

![PIC27](.\PIC\PIC27.png)

![PIC28](.\PIC\PIC28.png)

##### 具体流程

流程讲解思路：
• 以前述kitti中实现原理为基础，此处只是多了点云特征的约束；
• 只介绍可借鉴的内容，因此不介绍bias、外参初始化和外参优化等内容。

![PIC29](.\PIC\PIC29.jpg)

![PIC30](.\PIC\PIC30.jpg)

![PIC31](.\PIC\PIC31.jpg)

![PIC32](.\PIC\PIC32.png)

![PIC33](.\PIC\PIC33.jpg)

![PIC34](.\PIC\PIC34.jpg)

![PIC35](.\PIC\PIC35.jpg)

![PIC36](.\PIC\PIC36.jpg)

![PIC37](.\PIC\PIC37.jpg)

![PIC38](.\PIC\PIC38.jpg)

![PIC39](.\PIC\PIC39.jpg)

![PIC40](.\PIC\PIC40.jpg)

![PIC41](.\PIC\PIC41.jpg)

![PIC42](.\PIC\PIC42.jpg)

![PIC43](.\PIC\PIC43.jpg)

![PIC44](.\PIC\PIC44.jpg)

![PIC45](.\PIC\PIC45.jpg)

![PIC46](.\PIC\PIC46.jpg)

![PIC47](.\PIC\PIC47.jpg)

![PIC48](.\PIC\PIC48.jpg)

![PIC49](.\PIC\PIC49.jpg)

![PIC50](.\PIC\PIC50.jpg)

![PIC51](.\PIC\PIC51.jpg)

![PIC52](.\PIC\PIC52.jpg)

![PIC53](.\PIC\PIC53.jpg)

![PIC54](.\PIC\PIC54.jpg)

![PIC55](.\PIC\PIC55.jpg)

## 作业

按照课程讲述的模型，在提供的工程框架中，补全基于滑动窗口的融合定位方法的实现(整体思路本章第三小节已给出，代码实现可借鉴lio-mapping)，并分别与不加融合、EKF融合的效果做对比。
评价标准：
及格：补全代码，且功能正常。
良好：实现功能的基础上，性能在部分路段比EKF有改善。
优秀：由于基于滑窗的方法中，窗口长度对最终的性能有很大影响，请在良好的基础上，提供不同窗口长度下的融合结果，并对效果及原因做对比分析。


#### 附加题(不参与考核)： 

基于地图定位时，滑窗中是否要加入帧间里程计相对位姿约束，这是一个有争议的话题。若各位愿意，可在工程基础上对比两种方案的不同，并分析造成差异的原因。

## 环境配置

出现以下问题，是由于 make_unique 是c++ 14的新特性，需要在CMakelists.txt 中添加c++14 的编译指向。

```
lidar_localization/src/models/sliding_window/ceres_sliding_window.cpp:25:38: error: ‘make_unique’ is not a member of ‘std’
     config_.loss_function_ptr = std::make_unique<ceres::CauchyLoss>(new ceres::CauchyLoss(1.0));
```

解决， 在CMakelists.txt 添加

```
set(CMAKE_CXX_STANDARD 14)
```



注：protobuf要安装3.14.0版本的
