### 代码环境

在本地系统成功编译和运行，直接在src文件夹中更新替换这个包即可。

课程提供docker环境中报错：fmt库；加上已经在本地系统编译通过，重新生成了scan_context文件，在docker环境中编译还得重新生成，开发不是很便利，后面主要基于本地系统环境来完成。

### 运行

代码运行命令：

```
roslaunch lidar_localization lio_mapping.launch
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

上述三个ROS Service会生成所需的Map、trajectory Data与Scan Context Data. 分别位于:

```
Map: src/lidar_localization/slam_data/map
Scan Context Data: src/lidar_localization/slam_data/scan_context
trajectory Data: src/lidar_localization/slam_data/trajectory
```

evo工具运行命令：

```
evo_rpe kitti ground_truth.txt laser_odom.txt  -r trans_part --delta 100 --plot --plot_mode xyz
evo_rpe kitti ground_truth.txt optimized.txt -r trans_part --delta 100 --plot --plot_mode xyz
evo_ape kitti ground_truth.txt laser_odom.txt  -r full --plot --plot_mode xyz
evo_ape kitti ground_truth.txt optimized.txt -r full --plot --plot_mode xyz
```

