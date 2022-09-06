### 代码环境

在本地系统成功编译和运行，直接在src文件夹中更新替换这个包即可。

课程提供docker环境中报错：fmt库；加上已经在本地系统编译通过，重新生成了scan_context文件，在docker环境中编译还得重新生成，开发不是很便利，后面主要基于本地系统环境来完成。

### 运行

#### 地图建立

代码运行命令：

```
roslaunch lidar_localization mapping.launch
```

播放数据集命令：

```
rosbag  play kitti_lidar_only_2011_10_03_drive_0027_synced.bag
```

保存地图与Loop Closure Data：

```
rosservice call /optimize_map
rosservice call /save_map 
rosservice call /save_scan_context 
```

上述三个ROS Service会生成所需的Map与Scan Context Data. 分别位于:

```
Map: src/lidar_localization/slam_data/map
Scan Context Data: src/lidar_localization/slam_data/scan_context
```

evo工具运行命令：

evo_rpe：

```
evo_rpe kitti ground_truth.txt optimized.txt  -r trans_part --delta 100 --plot --plot_mode xyz
evo_rpe kitti ground_truth.txt laser_odom.txt  -r trans_part --delta 100 --plot --plot_mode xyz
```

evo_ape：

```
evo_ape kitti ground_truth.txt optimized.txt  -r full --plot --plot_mode xyz
evo_ape kitti ground_truth.txt laser_odom.txt  -r full --plot --plot_mode xyz
```

#### 全局定位

代码运行命令：

```
roslaunch lidar_localization matching.launch
```

播放数据集命令：

```
rosbag  play kitti_lidar_only_2011_10_03_drive_0027_synced.bag
```

### 修改参数

config\matching\matching.yaml

1. scan_context_path和map_path为全局定位需要的路径
2. GNSS_flag为0时为建图模式，需要记录下此时的地图原点经纬高（打印出来了）；为1时为定位模式。
3. origin_latitude、origin_longitude、origin_altitude为地图原点经纬高，需要自己填进去。
4. matching_flag为1时为基于GNSS的定位，为0时为基于Scan_context的定位。

```
# data storage:
# a. scan context:
scan_context_path: /home/qjs/code/ROS_Localization/shenlan/04/global_localization_chapter4_ausim_ws/src/lidar_localization/slam_data/scan_context   
# b. global map:
map_path: /home/qjs/code/ROS_Localization/shenlan/04/global_localization_chapter4_ausim_ws/src/lidar_localization/slam_data/map/filtered_map.pcd

# 回环检测:
loop_closure_method: scan_context # 选择回环检测方法, 目前支持scan_context

# 匹配
registration_method: NDT   # 选择点云匹配方法，目前支持：NDT 

# 当前帧
# no_filter指不对点云滤波，在匹配中，理论上点云越稠密，精度越高，但是速度也越慢
# 所以提供这种不滤波的模式做为对比，以方便使用者去体会精度和效率随稠密度的变化关系
frame_filter: voxel_filter # 选择当前帧点云滤波方法，目前支持：voxel_filter、no_filter

# 局部地图
# 局部地图从全局地图切割得到，此处box_filter_size是切割区间
# 参数顺序是min_x, max_x, min_y, max_y, min_z, max_z
box_filter_size: [-50.0, 50.0, -50.0, 50.0, -50.0, 50.0]
local_map_filter: voxel_filter # 选择滑窗地图点云滤波方法，目前支持：voxel_filter、no_filter

# 全局地图
global_map_filter: voxel_filter # 选择滑窗地图点云滤波方法，目前支持：voxel_filter、no_filter


# 各配置选项对应参数
## ScanContext params:
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
## 匹配相关参数
NDT:
    res : 1.0
    step_size : 0.1
    trans_eps : 0.01
    max_iter : 30
## 滤波相关参数
voxel_filter:
    global_map:
        leaf_size: [0.9, 0.9, 0.9]
    local_map:
        leaf_size: [0.5, 0.5, 0.5]
    frame:
        leaf_size: [1.5, 1.5, 1.5]

GNSS_flag: 1   #1为定位，0为建图  
matching_flag: 1  #1为GNSS，0为scan_context    
origin_latitude: 22.529812
origin_longitude: 113.93311
origin_altitude: 3.4457
```

