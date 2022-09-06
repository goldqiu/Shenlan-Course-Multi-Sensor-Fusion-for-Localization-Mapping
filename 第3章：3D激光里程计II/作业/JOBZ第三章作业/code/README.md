### 代码环境

在本地系统成功编译和运行，直接在src文件夹中更新替换这个包即可。课程提供docker环境中报错：fmt库

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

### 修改参数

config\front_end.yaml

只需要修改求导方式和参数块定义方式

```
scan_registration:
    subscriber:
        velodyne:
            topic_name: /kitti/velo/pointcloud
            frame_id: velo_link
            queue_size: 100
    publisher:
        filtered:
            topic_name: /kitti/velo/pointcloud/filtered
            frame_id: velo_link
            queue_size: 100
        sharp:
            topic_name: /kitti/velo/pointcloud/sharp
            frame_id: velo_link
            queue_size: 100
        less_sharp:
            topic_name: /kitti/velo/pointcloud/less_sharp
            frame_id: velo_link
            queue_size: 100
        flat:
            topic_name: /kitti/velo/pointcloud/flat
            frame_id: velo_link
            queue_size: 100
        less_flat:
            topic_name: /kitti/velo/pointcloud/less_flat
            frame_id: velo_link
            queue_size: 100
        removed:
            topic_name: /kitti/velo/pointcloud/removed
            frame_id: velo_link
            queue_size: 100
    param:
        # used later for motion compensation:
        scan_period: 0.10
        # num. lidar scans:
        num_scans: 64
        # only measurements taken outside this range is kept:
        min_range: 5.0
        # neighborhood size for curvature calculation:
        neighborhood_size: 5
        # num. sectors for feature point generation:
        num_sectors: 6
        # feature point filters:
        filter:
            surf_less_flat:
                leaf_size: [0.2, 0.2, 0.2]
front_end:
    publisher:
        odometry:
            topic_name: /odometry/lidar/scan_to_scan
            frame_id: /map
            child_frame_id: /velo_link
            queue_size: 100
        path:
            topic_name: /trajectory/lidar/scan_to_scan
            frame_id: /map
            queue_size: 100
    param:
        # kdtree nearest-neighborhood thresh:
        distance_thresh: 25.0
        # lidar scan neighborhood thresh:
        scan_thresh: 2.50

grade_way: autograde    
#autograde为自动求导，analytic_grade为解析式求导

param_block: ceres_defined
#user_defined为自定义参数块，ceres_defined为ceres定义参数块。
```

