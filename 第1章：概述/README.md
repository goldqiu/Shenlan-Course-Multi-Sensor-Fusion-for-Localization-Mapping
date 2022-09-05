# 第1章：概述

## 课程内容

源码：01-introduction.zip

kitti数据集：kitti_lidar_only_2011_10_03_drive_0027_synced.zip

视频：第1节-课程概述.mp4

PPT：多传感器融合定位-第1讲 V2.pdf

### 课程笔记

讲师知乎：https://www.zhihu.com/people/ren-gan-16

企业缺有工程经验的高端人才，项目不在多在于精，更看重工程实践和思维能力。

项目开发思路：先run起来一个demo，然后不断优化它。

### 环境搭建

#### 课程提供的docker环境

参考github:

https://github.com/kahowang/sensor-fusion-for-localization-and-mapping/tree/main/%E7%AC%AC%E5%85%AB%E7%AB%A0%20%E5%9F%BA%E4%BA%8E%E6%BB%A4%E6%B3%A2%E7%9A%84%E8%9E%8D%E5%90%88%E6%96%B9%E6%B3%952/%E4%BD%9C%E4%B8%9A%E4%BB%A3%E7%A0%81%E6%A1%86%E6%9E%B6/sensor-fusion-for-localization-and-mapping/docker

或者在课程提供的docker文件中查看readme。

#### 自建docker环境（和本地系统环境搭建一致）

参考：https://github.com/electech6/LVI-SAM_detailed_comments

1. 拉取镜像

```
docker pull liangjinli/slam-docker:v1.2
```

新建文件夹（用于挂载docker环境）：/home/qjs/code/localization_docker

2. 第一次运行镜像：

docker run -v /home/qjs/code/localization_docker/:/home/ --net=host -it liangjinli/slam-docker:v1.2 /bin/bash

3. 后续运行：

docker start  CONTAINER-ID

4. 进入容器：

docker exec -it  CONTAINER-ID /bin/bash

5. 关闭容器：

docker stop CONTAINER-ID

6. 安装课程所需环境：

参考：https://github.com/kahowang/sensor-fusion-for-localization-and-mapping/tree/main/%E7%AC%AC%E4%B8%80%E7%AB%A0%20%E6%A6%82%E8%BF%B0

https://github.com/AlexGeControl/Sensor-Fusion-for-Localization-Courseware

7. 第三方库文件见第三方库文件夹

8. 查看容器ID：

docker ps   -a

9. 查看镜像ID：

docker images  

10. 保存镜像：

sudo docker commit -m="add config" -a="qjs" 4eac4bdb61be（CONTAINER ID） liangjinli/slam-docker:v1.3（自定义）    

保存容器为新的镜像版本，这一步很重要，系统每次配置完环境需要重新提交，更新镜像版本，下次运行镜像，开启容器时，选择最新的版本，如果不更新就白装环境了，但是很方便版本管理。

11. 保存到本地：

用容器保存和导入
docker export f299f501774c（CONTAINER ID） > hangger_server.tar
docker import - new_hangger_server < hangger_server.tar
用镜像保存和导入
docker save 0fdf2b4c26d3（IMAGE ID） > hangge_server.tar
docker load < hangge_server.tar

ps：

1. 这种方法暂时还没弄可视化，但是是可以弄的，暂时使用可以用本地系统中的rviz显示。
2. 课程环境的搭建和本地系统环境搭建是一致的，故可以参考下面。
2. 该镜像装了gtsam，需要卸载后重装

#### 本地系统环境搭建

1. 安装Ubuntu18.04和ROS开发环境，参考博客：

[goldqiu：三.开发记录之移动硬盘装ubuntu系统的配置、环境、各类软件安装和备份等](https://zhuanlan.zhihu.com/p/432915999)

[goldqiu：四.开发记录之ubuntu系统安装ROS和开发环境](https://zhuanlan.zhihu.com/p/433535056)

[goldqiu：五.开发记录之ubuntu系统安装各个软件](https://zhuanlan.zhihu.com/p/455801206)

2. 安装docker

参考官方文档：

https://docs.docker.com/engine/install/ubuntu/

测试：

sudo docker pull hello-world

sudo docker run hello-world

将当前用户加入Docker Group：

为了能在非`sudo`模式下使用`Docker`, 需要将当前用户加入`Docker Group`.

执行命令:

```
sudo usermod -aG docker $USER
```

为了使上述变更生效，请先Logout，再Login

3. 安装课程环境，参考:

https://github.com/kahowang/sensor-fusion-for-localization-and-mapping/tree/main/%E7%AC%AC%E4%B8%80%E7%AB%A0%20%E6%A6%82%E8%BF%B0

https://github.com/AlexGeControl/Sensor-Fusion-for-Localization-Courseware

第三方库文件见第三方库文件夹

可能需要安装：pcl-ros

sudo apt-get install ros-melodic-pcl-ros

ps：

1. GeographicLib在项目工程中有，可以不用装。
2. glog下载0.3.5版本源码安装。
3. gflag直接github下载最新源码安装。

报错：

```
undefined reference to `google::FlagRegisterer::FlagRegisterer<bool>
```

cmakelists添加

```
find_package(gflags REQUIRED)
target_link_libraries(gflags)
```

4. 安装sophus报错：

```
/home/qjs/pack/Sophus/sophus/so2.cpp:32:26: error: lvalue required as left operand of assignment
   unit_complex_.real() = 1.;
                          ^~
/home/qjs/pack/Sophus/sophus/so2.cpp:33:26: error: lvalue required as left operand of assignment
   unit_complex_.imag() = 0.;
                          ^~
```

找到如下代码：

```
SO2::SO2()
{
  unit_complex_.real() = 1.;
  unit_complex_.imag() = 0.;
}
```

将其修改为

```
SO2::SO2()
{
  unit_complex_.real(1.);
  unit_complex_.imag(0.);
}
```

5. 安装ceres报错：undefined glflags_shared

cmake -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=ON -DINSTALL_HEADERS=ON -DINSTALL_SHARED_LIBS=ON -DINSTALL_STATIC_LIBS=ON -DGFLAGS_NAMESPACE=google ..
然后再编译

## 作业

![Screenshot from 2022-08-01 22-09-01](E:\AUSIM\Localization\SynologyDrive\第1章：概述\作业\Screenshot from 2022-08-01 22-09-01.png)

1. 代码见文件夹。
2. 使用kitti_lidar_only_2011_10_03_drive_0027_synced.bag这个数据集。
3. 运行：roslaunch lidar_localization hello_kitti.launch

## 实验室实车实现	

![Screenshot from 2022-08-02 10-53-04](E:\AUSIM\Localization\SynologyDrive\第1章：概述\实验室实车实现\Screenshot from 2022-08-02 10-53-04.png)

1. 代码中适配了实验室小车数据集，但没有进行真正的传感器软时间同步，暂时没有什么影响，后面可以做。
2. 使用的数据集在NAS的路径为：Ausim_Public\AMR平台数据\项目自录数据集\建图定位汇总\建图-小地图-可用于定位\20220715_2022-07-15-18-51-38.bag
3. 代码见文件夹。
4. 运行：roslaunch lidar_localization hello_kitti.launch

