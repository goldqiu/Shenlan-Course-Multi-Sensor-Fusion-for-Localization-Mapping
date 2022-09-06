# 第3章：3D激光里程计II

## 课程内容

代码、PPT、视频见文件夹

### 课程笔记

本质：拿特征做里程计的推算。

参考github：

https://github.com/AlexGeControl/Sensor-Fusion-for-Localization-Courseware

https://github.com/kahowang/sensor-fusion-for-localization-and-mapping

#### 点线面几何基础：

![pic1](.\PIC\pic1.jpg)

![pic2](.\PIC\pic2.jpg)

![pic3](.\PIC\pic3.jpg)

![pic4](.\PIC\pic4.jpg)

#### 点云线面特征提取:

![pic5](.\PIC\pic5.jpg)

![pic6](.\PIC\pic6.jpg)

参考文献：LOAM: Lidar Odometry and Mapping in Real-time Ji Zhang and Sanjiv Singh 

推荐博客：https://blog.csdn.net/robinvista/article/details/104379087

![pic7](.\PIC\pic7.jpg)

在第k+1帧中找到一个线特征点（曲率最大），转换到k帧坐标系下（根据运动模型或者其他传感器猜测得到），然后在第k帧中找与k+1帧中特征点最近的一个点，然后再找相邻的一个点，组成一条线，那么第k+1帧的点到第k帧的线就有了，可以构建优化问题了。

![pic8](.\PIC\pic8.jpg)

面特征也一样，先在第k帧搜索离第k+1帧面点（曲率最小）最近的一个点，然后在第k帧是找同根线的一个点和相邻线的一个点，三点构建平面，然后构建点面残差方差，进行优化求解。

![pic9](.\PIC\pic9.jpg)

整个优化问题的核心是求得残差关于待求变量的雅可比矩阵。

#### 基于线面特征的位姿优化

![pic10](.\PIC\pic10.jpg)

![pic11](.\PIC\pic11.jpg)

#### 位姿优化代码实现

##### ceres 基础知识

![pic12](.\PIC\pic12.jpg)

![pic13](.\PIC\pic13.jpg)

![pic14](.\PIC\pic14.jpg)

![pic15](.\PIC\pic15.jpg)

![pic16](.\PIC\pic16.jpg)

##### 自动求导与解析求导的对比

• 自动求导实现方便，但效率会比解析求导低 (比较 A-LOAM 和 F-LOAM )； 

• 实际使用中，能够自动求导且效率没有形成障碍的，优先使用自动求导； 

• 除这两种方法外，还有数值求导（SLAM问题中不常见，不过多介绍）

##### 自动求导实现位姿优化(A-LOAM)

![pic17](.\PIC\pic17.jpg)

![pic18](.\PIC\pic18.jpg)

##### 解析求导实现位姿优化(F-LOAM)

![pic19](.\PIC\pic19.jpg)

![pic20](.\PIC\pic20.jpg)

#### 相关开源里程计

#####  基于特征的里程计实现流程

![pic21](.\PIC\pic21.jpg)

只做帧间匹配的后果：

1. 雷达角分辨率不够、频率低，导致下一帧没有特征关联。
2. 精度也不够，两帧之间的关联特征很少，优化的精度不够。

![pic22](.\PIC\pic22.jpg)

A-LOAM主要特点
1) 去掉了和IMU相关的部分
2) 使用Eigen（四元数）做位姿转换，简化了代码
3) 使用ceres做迭代优化，简化了代码，但降低了效率
代码：https://github.com/HKUST-Aerial-Robotics/A-LOAM

 F-LOAM主要特点
1) 整体和ALOAM类似，只是使用残差函数的雅可比使用的是解析式求导
代码：https://github.com/wh200720041/floam

原始loam是没有帧的概念的，帧和地图做匹配，地图已经是无序的点云了，是做线和平面拟合来构建残差。没有保存每个关键帧的位姿和点云，进行后端优化后的调整，这是比较不足的。 	

## 作业

![pic23](.\PIC\pic23.jpg)

代码和文档见文件夹。

注：

由于 make_unique 是c++ 14的新特性，需要在CMakelists.txt 中添加c++14 的编译指向：

在CMakelists.txt 添加

```
set(CMAKE_CXX_STANDARD 14)
```

参考博客：

[slam中ceres的常见用法总结](https://blog.csdn.net/qq_42700518/article/details/105898222)

[Ceres详解（一） Problem类](https://blog.csdn.net/weixin_43991178/article/details/100532618)

[Ceres详解（二） CostFunction](https://blog.csdn.net/weixin_43991178/article/details/100534230)

[Ceres（二）LocalParameterization参数化](https://blog.csdn.net/hzwwpgmwy/article/details/86490556)

[优化库——ceres（二）深入探索ceres::Problem](https://blog.csdn.net/jdy_lyy/article/details/119360492)

[[ceres-solver\] From QuaternionParameterization to LocalParameterization](https://www.cnblogs.com/JingeTU/p/11707557.html)

[基于线面特征的激光里程计](https://zhuanlan.zhihu.com/p/348195351)



## 实验室实车实现	

代码和文档见文件夹。
