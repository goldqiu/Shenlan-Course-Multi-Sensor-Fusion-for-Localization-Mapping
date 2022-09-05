### 代码环境

在本地系统和课程提供docker环境中都成功编译和运行，直接在src文件夹中更新替换这个包即可。

### 运行

代码运行命令：

```
roslaunch lidar_localization front_end.launch
```

evo工具运行命令：

evo_rpe：

```
evo_rpe kitti ground_truth.txt laser_odom.txt -r trans_part --delta 100 --plot --plot_mode xyz
```

evo_ape：

```
evo_ape kitti ground_truth.txt laser_odom.txt -r full --plot --plot_mode xyz
```

播放数据集命令：

```
rosbag  play kitti_lidar_only_2011_10_03_drive_0027_synced.bag
```

保存地图命令：

```
rosservice call /save_map
```

### 修改参数

只需要修改数据存放路径、匹配方法，其他一般不需要修改。

```
data_path: /home/qjs/code/ROS_Localization/shenlan/02/global_localization_chapter2_ws/src/lidar_localization   # 数据存放路径

# 匹配
# TODO: implement your custom registration method and add it here
registration_method: SICP    # 选择点云匹配方法，目前支持：ICP, ICP_SVD, NDT, SICP,NDT_CPU

# 局部地图
key_frame_distance: 2.0 # 关键帧距离
local_frame_num: 20  # localmap中关键帧的数量
local_map_filter: voxel_filter # 选择滑窗地图点云滤波方法，目前支持：voxel_filter

# rviz显示
display_filter: voxel_filter # rviz 实时显示点云时滤波方法，目前支持：voxel_filter

# 当前帧
frame_filter: voxel_filter # 选择当前帧点云滤波方法，目前支持：voxel_filter

## 滤波相关参数
voxel_filter:
    local_map:
        leaf_size: [0.6, 0.6, 0.6]
    frame:
        leaf_size: [1.3, 1.3, 1.3]
    display:
        leaf_size: [0.5, 0.5, 0.5]

# 各配置选项对应参数
## 匹配相关参数
ICP:
    max_corr_dist : 1.2  #最近邻点对的距离阈值
    trans_eps : 0.01   #变换矩阵最大容差
    euc_fitness_eps : 0.36     #欧式距离最大容差
    max_iter : 30   #最大迭代次数
ICP_SVD:
    max_corr_dist : 0.5
    trans_eps : 0.01
    euc_fitness_eps : 0.36
    max_iter : 10
# ICP_SVD:
#     max_corr_dist : 1.2
#     trans_eps : 0.01
#     euc_fitness_eps : 0.36
#     max_iter : 10
NDT:
    res : 1.0    #voxel resolution 体素栅格的分辨率或大小
    step_size : 0.1  # 梯度下降的步长，越大下降越快，但是容易over shoot陷入局部最优
    trans_eps : 0.01 # 变换矩阵最大容差，一旦两次转换矩阵小于 trans_eps  退出迭代
    max_iter : 30  #   最大迭代次数
# NDT_CPU:
#     res : 0.8                 # volex  resolution
#     step_size : 0.1    # 梯度下降的步长，越大下降越快，但是容易over shoot陷入局部最优
#     trans_eps : 0.01    # 最大容差，一旦两次转换矩阵小于 trans_eps  退出迭代
#     max_iter : 30         #   最大迭代次数
NDT_CPU:
    res : 1.0                 # volex  resolution
    step_size : 0.1    # 梯度下降的步长，越大下降越快，但是容易over shoot陷入局部最优
    trans_eps : 0.01    # 最大容差，一旦两次转换矩阵小于 trans_eps  退出迭代
    max_iter : 30         #   最大迭代次数
SICP:
     p : 1.0
     mu : 10.0
     alpha : 1.2
     max_mu : 1e5
     max_icp : 100
     max_outer : 100
     max_inner : 1
     stop : 1e-5
```

