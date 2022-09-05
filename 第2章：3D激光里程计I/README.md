# 第2章：3D激光里程计I

## 课程内容

代码、PPT、视频见文件夹

### 课程笔记

参考github：

https://github.com/AlexGeControl/Sensor-Fusion-for-Localization-Courseware

https://github.com/kahowang/sensor-fusion-for-localization-and-mapping

#### Lidar的分类：

机械旋转激光雷达(如vlp16)，固态激光雷达(如Livox Mid-40)

不同点：

a. 视角范围不相同

b. 扫描工作方式不同

**机械旋转激光雷达:** 

一般是多个激光束同时发射，并绕固定轴旋转，来探测三维环境。

• 激光雷达传感器向周围环境发射脉冲光波； 

• 这些脉冲碰撞到周围物体反弹并返回传感器； 

• 传感器使用每个脉冲返回到传感器所花费的时间来计算其传播的距离； 

• 每秒重复数百万次此过程，将创建精确的实时3D环境地图

缺点：远处的激光点之间的间隔较大

**固态激光雷达：**

非重复扫描，它的扫描方式是梅花瓣状的。 每一帧激光点并不重合，这意味着在Livox Mid-40在静态放置的状态下，经过多次扫描可以获得比机械旋转激光更稠密的点云。

#### 建图的流程：

![pic1](.\PIC\pic1.jpg)

cartographer运算量比较大，适合于小场景，不是3dSLAM的主流方案。

#### 前端里程计方案—基于直接匹配

##### 点到点ICP-基于SVD

基于解析式求解

![pic2](.\PIC\pic2.jpg)

![pic3](.\PIC\pic3.jpg)

首先通过最近邻查找（可用KD-Tree）到离当前点云最近的点云块，就确定了两个点集，且一一对应。然后计算求解点集之间的旋转平移关系。然后旋转过去，再进行最近邻查找，再确定两个点集，再求解旋转平移关系，不断迭代到一个阈值后得到最终的旋转平移关系。最终点集之间的距离越小则说明旋转平移关系越准确。

**公式推导：**

见参考文献中的icp公式推导。

##### ICP优化法

优化任务的目标：

![pic7](.\PIC\pic7.jpg)

方程中， f(x)是残差函数，在实际使用中，它可以 代表任何方式得到的残差，比如 ICP 中点到点之间的 距离、融合中预测与观测之间的误差等等。

迭代方法的思路:

当方程形式复杂时，无法直接求出解析解，因 此需要使用迭代方法，找到最优解，步骤为：

![pic8](.\PIC\pic8.jpg)

**最速下降法：**

对损失函数进行泰勒展开：

![pic5](.\PIC\pic5.jpg)

只保留一阶泰勒展开，并沿梯度的反方向取增量。

优点：迭代方便、计算简单 

缺点：

a. 一阶近似精度有限，容易走出锯齿形状 

b. 越接近目标值，步长越小，前进越慢

**牛顿法**

（考虑变化率）：保留二阶泰勒展开结果

![pic6](.\PIC\pic6.jpg)

优点 ：

a. 对原函数的近似更精确，每一步的收敛更加准确 

b. 收敛速度快 

缺点：

需要计算H矩阵，在优化规模较大时，不容易做到

**高斯牛顿法**

对残差函数进行泰勒展开，而非损失函数

![pic9](.\PIC\pic9.jpg)

![pic10](.\PIC\pic10.jpg)

![pic11](.\PIC\pic11.jpg)

![pic4](.\PIC\pic4.jpg)

法空间采样：能够采样出更适合匹配的纹理性的点

参考文献：

1) Arun, K. Somani, Thomas S. Huang, and Steven D. Blostein. "Least-squares fitting of two 3-D point sets." IEEE Transactions on pattern analysis and machine intelligence 5 (1987): 698-700. 

2) https://igl.ethz.ch/projects/ARAP/svd_rot.pdf

http://www2.imm.dtu.dk/pubdb/edoc/imm3215.pdf

参考博客：

[深蓝学院 《多传感器融合定位》 第1章作业](https://blog.csdn.net/hitljy/article/details/109227799?spm=1001.2014.3001.5501)

[Geyao 前端里程计代码](https://github.com/AlexGeControl/Sensor-Fusion/tree/master/workspace/assignments/01-lidar-odometry)

[雷达系列论文翻译（二）：Least-Squares Rigid Motion Using SVD](https://blog.csdn.net/weixin_39061796/article/details/119861178)

##### NDT

基于概率的匹配

假设A点云静止不动，空间划分栅格。B点云通过R t旋转到A点云，正好落在栅格上，则说明匹配上了。最终是两个点云的分布很接近。

ICP计算的是点对点，NDT是考虑高斯分布计算联合概率，求B点云落在A点云栅格上的最大概率，此时两个点云概率分布最近似，对应的R,t就是我们要求的。

![pic12](.\PIC\pic12.jpg)

![pic13](.\PIC\pic13.jpg)

![pic14](.\PIC\pic14.jpg)

![pic15](.\PIC\pic15.jpg)

![pic16](.\PIC\pic16.jpg)

八叉树：没有点的地方不给划分栅格，少点的地方栅格尺寸大些。

增加鲁棒性：暴力划分栅格，比如对于一棵树，可能会被划分成一半；这可以用插值的方法优化。

参考论文：

1. Scan Registration using Segmented Region Growing NDT. Das A, Waslander SL. 2014. 

2. 3D Scan Registration Using the Normal Distributions Transform with Ground Segmentation and Point Cloud Clustering. Das A, Waslander SL. 2013. 
3. Scan Registration with Multi-Scale K-Means Normal Distributions Transform. Das A, Waslander SL. 2012.

[NDT 公式推导及源码解析（1）](https://blog.csdn.net/u013794793/article/details/89306901)

[NDT 公式推导及源码解析（2）](https://blog.csdn.net/u013794793/article/details/89321792)

#### 点云畸变处理

**产生原因** 

一帧点云：通常指雷达内部旋转一周扫描得到的点的集合 

优点：有足够数量的点云才能进行匹配，且一周正好是周围环境的完整采集。

缺点：每个激光点的坐标都是相对于雷达的，雷达运动时，不同激光点的坐标原点会不同

![pic17](.\PIC\pic17.jpg)

![pic18](.\PIC\pic18.jpg)

![pic19](.\PIC\pic19.jpg)

使用方法：

1) 正常雷达驱动输出的数据(如velodyne驱动)均可按此逻辑进行补偿。 

2) 角速度和线速度输入，可以使用imu、编码器等外接传感器，也可以使用slam的相对位姿，后者效果会稍差。 

特别事项： 作业工程中提供的畸变补偿方法与ppt中提供的代码不同， 原因是kitti提供的数据打乱了点云排列格式。 由于这种问题在实际工程中不会出现，且该方法理解较为复杂，故此处只对思路做大致介绍，可跳过对作业工程中该部分代码的学习。

![pic20](.\PIC\pic20.jpg)

可以用IMU测角速度，轮速计测线速度，IMU线速度不是很准。

进阶思考 1) 上述方法以一帧点云起始时间作为该帧点云时间，若以终止时间或中间时刻作为点云时间，畸变补偿方法应如何变化？ 

2) 若雷达内部是逆时针旋转，而不是顺时针旋转，畸变补偿方法应如何变化？ 

若想对各种情况变化做出适应性改动，须理解雷达运动造成点云畸变的核心机理才 行，不要死记方法。

#### KITTI数据集简介

硬件组成： 

1) 一个64线激光雷达，在车顶的正中心 

2) 两个彩色摄像头和两个黑白摄像头，在雷达两侧。 

3) 一个组合导航系统（OXTS RT 3003），在雷达左后 方。它可以输出RTK/IMU组合导航结果，包括经纬度和姿态，同时也输出IMU原始数据。

安装关系：
图中所示的所有安装关系，都可以在数据集提供标定文件中找到，可直接使用。

#### 里程计工程框架实现

核心思想：
1) 通过类的封装，实现模块化
2) 把ros流程与c++内部实现分开，使流程清晰
3) 基于c++多态，实现高可扩展性

![pic21](.\PIC\pic21.jpg)

#### 里程计精度评价 

以组合导航的结果为真值，使用evo工具进行里程计精度评价 

1) 安装evo pip install evo --upgrade --no-binary evo 

2) 使用evo计算轨迹误差 

a. 分段统计精度 

evo_rpe kitti ground_truth.txt laser_odom.txt -r  trans_part --delta 100 --plot --plot_mode xyz 

b. 计算整体轨迹误差 

evo_ape kitti ground_truth.txt laser_odom.txt -r full -- plot --plot_mode xyz

EVO评价数据有两种模式，对应的指令分别是 evo_rpe 和 evo_ape ，前者评价的是每段距离内的误差，后者评价的是绝对误差随路程的累计。

其中–delta 100表示的是每隔100米统计一次误差，这样统计的其实就是误差的百分比，和kitti的odometry榜单中的距离误差指标就可以直接对应了。

## 作业

代码和文档见文件夹。

ICP实现可参考：https://github.com/tttamaki/SICP-test

## 实验室实车实现	

代码和文档见文件夹。

## 第二章答疑

1. SICP的效果的确不佳，会飘，且效率低。
2. PCL-ICP的效果也不好，会飘。
3. 归一化的必要：transformation是多个旋转矩阵连乘得到的，由于计算机精度的影响可能使得旋转矩阵不满足其性质，有可能它行列式不为1或不对称。
4. 求解svd时就要判断检查VU相乘的行列式是1还是-1，-1是icp出现退化的时候才会有，参考论文有提及。本身u和v都是正交阵，乘完求出结果也是正交阵，但只有行列式为1的正交阵才是旋转矩阵。