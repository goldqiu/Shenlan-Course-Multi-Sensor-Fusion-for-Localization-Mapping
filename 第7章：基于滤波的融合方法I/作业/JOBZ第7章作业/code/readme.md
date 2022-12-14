### 代码环境

在本地系统成功编译和运行，直接在src文件夹中更新替换这个包即可。

课程提供docker环境中报错：fmt库；加上已经在本地系统编译通过，重新生成了scan_context文件，在docker环境中编译还得重新生成，开发不是很便利，后面主要基于本地系统环境来完成。

### 运行

代码运行命令：

```
roslaunch lidar_localization kitti_localization.launch 
```

播放数据集命令：

```
rosbag  play kitti_lidar_only_2011_10_03_drive_0027_synced.bag
```

保存里程计：

```
rosservice call /save_odometry
```

数据位于:

```
src/lidar_localization/slam_data/trajectory
```

evo工具运行命令：

```
# a. laser  输出评估结果，并以zip的格式存储:
evo_ape kitti ground_truth.txt laser.txt -r full --plot --plot_mode xy  --save_results ./laser.zip
# b. fused 输出评估结果，并以zip的格式存储:
evo_ape kitti ground_truth.txt fused.txt -r full --plot --plot_mode xy  --save_results ./fused.zip
#c. 比较 laser  fused  一并比较评估
evo_res  *.zip -p
```

### 修改参数

config\filtering\kitti_filtering.yaml

1. scan_context_path和map_path为全局定位需要的路径
2. process和measurement对应的参数分别为卡尔曼滤波器的Q、R参数
3. bias_flag: true为考虑随机游走，false为不考虑

```
# 全局地图
map_path: /home/qjs/code/ROS_Localization/shenlan/04/global_localization_chapter4_ws/src/slam_data/map/filtered_map.pcd
global_map_filter: voxel_filter # 选择滑窗地图点云滤波方法，目前支持：voxel_filter、no_filter

# 局部地图
# 局部地图从全局地图切割得到，此处box_filter_size是切割区间
# 参数顺序是min_x, max_x, min_y, max_y, min_z, max_z
box_filter_size: [-150.0, 150.0, -150.0, 150.0, -150.0, 150.0]
local_map_filter: voxel_filter # 选择滑窗地图点云滤波方法，目前支持：voxel_filter、no_filter

# 当前帧
# no_filter指不对点云滤波，在匹配中，理论上点云越稠密，精度越高，但是速度也越慢
# 所以提供这种不滤波的模式做为对比，以方便使用者去体会精度和效率随稠密度的变化关系
current_scan_filter: voxel_filter # 选择当前帧点云滤波方法，目前支持：voxel_filter、no_filter

# loop closure for localization initialization/re-initialization:
loop_closure_method: scan_context # 选择回环检测方法, 目前支持scan_context
scan_context_path: /home/qjs/code/ROS_Localization/shenlan/04/global_localization_chapter4_ws/src/slam_data/scan_context   

# 匹配
registration_method: NDT   # 选择点云匹配方法，目前支持：NDT 

# select fusion method for IMU-GNSS-Odo-Mag, available methods are:
#     1. error_state_kalman_filter
fusion_method: error_state_kalman_filter
# select fusion strategy for IMU-GNSS-Odo-Mag, available methods are:
#     1. pose
fusion_strategy: pose

# 各配置选项对应参数
## a. point cloud filtering:
voxel_filter:
    global_map:
        leaf_size: [0.9, 0.9, 0.9]
    local_map:
        leaf_size: [0.5, 0.5, 0.5]
    current_scan:
        leaf_size: [1.5, 1.5, 1.5]
## b. scan context:
scan_context:
    # a. ROI definition:
    max_radius: 80.0
    max_theta: 360.0
    # b. resolution:
    num_rings: 20
    num_sectors: 60
    # c. ring key indexing interval:
    indexing_interval: 1
    # d. min. key frame sequence distance:
    min_key_frame_seq_distance: 100
    # e. num. of nearest-neighbor candidates to check:
    num_candidates: 5
    # f. sector key fast alignment search ratio:
    #   avoid brute-force match using sector key
    fast_alignment_search_ratio: 0.1
    # g. scan context distance threshold for proposal generation:
    #   0.4-0.6 is good choice for using with robust kernel (e.g., Cauchy, DCS) + icp fitness threshold 
    #   if not, recommend 0.1-0.15
    scan_context_distance_thresh: 0.15
## c. frontend matching
NDT:
    res : 1.0
    step_size : 0.1
    trans_eps : 0.01
    max_iter : 30
## d. Kalman filter for IMU-lidar-GNSS fusion:
## d.1. Error-State Kalman filter for IMU-GNSS-Odo fusion:
error_state_kalman_filter:
    earth:
        # gravity can be calculated from https://www.sensorsone.com/local-gravity-calculator/ using latitude and height:
        gravity_magnitude: 9.80943
        # rotation speed, rad/s:
        rotation_speed: 7.292115e-5
        # latitude:
        latitude:   48.9827703173
    covariance:
        prior:
            pos: 1.0e-6
            vel: 1.0e-6
            ori: 1.0e-6
            epsilon: 1.0e-6
            delta: 1.0e-6
        process:
            gyro: 1.0e-2
            accel: 2.5e-1
            bias_accel: 2.5e-1
            bias_gyro: 1.0e-2
            bias_flag: true
        measurement:
            pose:
                pos: 1.0e-7
                ori: 1.0e-7
            pos: 1.0e-7
            vel: 2.5e-6
    motion_constraint: 
        activated: true
        w_b_thresh: 0.13
```

