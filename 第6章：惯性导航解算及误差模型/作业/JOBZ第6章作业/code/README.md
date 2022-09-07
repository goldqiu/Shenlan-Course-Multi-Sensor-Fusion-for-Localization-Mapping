### 代码环境

在本地系统成功编译和运行，直接在src文件夹中更新替换这个包即可。

### 运行

#### 解算方法更换

用宏定义：

```c++
#define  MedianMethod    //  use  euler  method    or  median method
```

#define  MedianMethod 则为中值法，注释掉则为欧拉法。

#### 课程提供数据

```
roslaunch imu_integration imu_integration.launch 
```

#### 仿真数据生成

```
roslaunch gnss_ins_sim recorder_gnss_ins_sim.launch
```

yaml文件更改：

FILE: src/gnss_ins_sim/config/recorder_gnss_ins_sim.yaml

```
# motion def:
motion_file: recorder_gnss_ins_sim_speedup_down.csv
# IMU params:
imu: 1
# sample frequency of simulated GNSS/IMU data:
sample_frequency:
    imu: 100.0
    gps: 10.0
# topic name:
topic_name_imu: /sim/sensor/imu
topic_name_gt:  /pose/ground_truth
# output rosbag path:
output_path: /home/qjs/code/ROS_Localization/shenlan/06/global_localization_chapter6_ws/src/data/gnss_ins_sim/recorder_gnss_ins_sim
# output name:
output_name: speedup_down.bag
```

需要更改文件输出路径、motion_file、output_name

#### 数据评估

```
roslaunch imu_integration imu_ins_gnss.launch
rosbag play xx.bag
可能需要 rosbag reindex xx.bag
```

最后evo评估：

```
evo_rpe tum gt.txt ins.txt -r full --delta 100 --plot --plot_mode xyz
```

