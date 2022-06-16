# FAST_LIO_SAM

## Front_end : fastlio2      Back_end : lio_sam

## Related worked 

1.[FAST-LIO2](https://github.com/hku-mars/FAST_LIO)为紧耦合的lio slam系统，因其缺乏前端，所以缺少全局一致性，参考lio_sam的后端部分，接入GTSAM进行后端优化。

2.[FAST_LIO_SLAM](https://github.com/gisbi-kim/FAST_LIO_SLAM)的作者kim在FAST-LIO2的基础上，添加SC-PGO模块，通过加入ScanContext全局描述子，进行回环修正,SC-PGO模块与FAST-LIO2解耦，非常方便，很优秀的工作。

3.[FAST_LIO_LC](https://github.com/yanliang-wang/FAST_LIO_LC)的作者yanliang-wang,在FAST_LIO_SLAM的基础上添加了：1.基于Radius Search 基于欧式距离的回环检测搜索，增加回环搜索的鲁棒性；2.回环检测的优化结果，更新到FAST-LIO2的当前帧位姿中，幷进行ikdtree的重构，进而更新submap。

## Contributions  

[FAST_LIO_SAM](https://github.com/kahowang/FAST_LIO_SAM)的主要贡献：

1.对比[FAST_LIO_SLAM](https://github.com/gisbi-kim/FAST_LIO_SLAM/tree/bf975560741c425f71811c864af5d35aa880c797) 与 [FAST_LIO_LC](https://github.com/yanliang-wang/FAST_LIO_LC) 使用外部接入的PGO回环检测模块进行后端优化 ，FAST_LIO_SAM 将LIO-SAM的后端GTSAM优化部分移植到FAST-LIO2的代码中，数据传输处理环节更加清晰。

2.增加关键帧的保存，可通过rosservice的指令对地图和轨迹进行保存。

3.FAST_LIO_SLAM中的后端优化，只使用了GPS的高层进行约束，GPS的高层一般噪声比较大，所以添加GPS的XYZ三维的postion进行GPS先验因子约束。

## Prerequisites

- Ubuntu 18.04 and ROS Melodic
- PCL >= 1.8 (default for Ubuntu 18.04)
- Eigen >= 3.3.4 (default for Ubuntu 18.04)
- GTSAM >= 4.0.0(tested on 4.0.0-alpha2)

## Build

```shell
cd YOUR_WORKSPACE/src
git clone https://github.com/kahowang/FAST_LIO_SAM.git
cd ..
catkin_make
```

## Quick test

### Loop clousre：

#### 1 .For indoor dataset 

dataset is from yanliang-wang ,[dataset](https://drive.google.com/file/d/1NGTN3aULoTMp3raF75LwMu-OUtzUx-zX/view?usp=sharing) which includes `/velodyne_points`(10Hz) and `/imu/data`(400Hz).

```shell
roslaunch fast_lio_sam mapping_velodyne16.launch
rosbag play  T3F2-2021-08-02-15-00-12.bag  
```

<p align ="center">
<img src = "./pic/car.gif "  alt ="car" width = 40%  height =30%; "/>
</p>

#### 2 .For outdoor dataset

dataset is from [LIO-SAM](https://github.com/TixiaoShan/LIO-SAM) **Walking dataset:** [[Google Drive](https://drive.google.com/drive/folders/1gJHwfdHCRdjP7vuT556pv8atqrCJPbUq?usp=sharing)]

```shell
roslaunch fast_lio_sam mapping_velodyne16_lio_sam_dataset.launch
rosbag  play  walking_dataset.bag
```

### 3.save_map

输入如下指令到terminal中，地图文件将会保存在应文件夹中

```shell
rosservice call /save_map "resolution: 0.0
destination: ''" 
success: True
```

### 4.save_poes

输入如下指令到terminal中，poes文件将会保存在相应文件夹中

```shell
rosservice call /save_pose "resolution: 0.0
destination: ''" 
success: False
```

evo 绘制轨迹

```shell
evo_traj kitti optimized_pose.txt without_optimized_pose.txt -p
```

| ![evo1](https://kaho-pic-1307106074.cos.ap-guangzhou.myqcloud.com/CSDN_Pictures/%E6%B7%B1%E8%93%9D%E5%A4%9A%E4%BC%A0%E6%84%9F%E5%99%A8%E8%9E%8D%E5%90%88%E5%AE%9A%E4%BD%8D/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E6%BF%80%E5%85%89%E9%87%8C%E7%A8%8B%E8%AE%A11evo1.png) | ![evo2](https://kaho-pic-1307106074.cos.ap-guangzhou.myqcloud.com/CSDN_Pictures/%E6%B7%B1%E8%93%9D%E5%A4%9A%E4%BC%A0%E6%84%9F%E5%99%A8%E8%9E%8D%E5%90%88%E5%AE%9A%E4%BD%8D/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E6%BF%80%E5%85%89%E9%87%8C%E7%A8%8B%E8%AE%A11evo2.png) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |

### 5.some config 

```shell
# Loop closure
loopClosureEnableFlag: true		      # use loopclousre or not 
loopClosureFrequency: 4.0                     # Hz, regulate loop closure constraint add frequency
surroundingKeyframeSize: 50                   # submap size (when loop closure enabled)
historyKeyframeSearchRadius: 1.5             # meters, key frame that is within n meters from current pose will be considerd for loop closure
historyKeyframeSearchTimeDiff: 30.0           # seconds, key frame that is n seconds older will be considered for loop closure
historyKeyframeSearchNum: 20                  # number of hostory key frames will be fused into a submap for loop closure
historyKeyframeFitnessScore: 0.3              # icp threshold, the smaller the better alignment

# visual iktree_map  
visulize_IkdtreeMap: true

# visual iktree_map  
recontructKdTree: true

savePCDDirectory: "/fast_lio_sam_ws/src/FAST_LIO_SAM/PCD/"        # in your home folder, starts and ends with "/". Warning: the code deletes "LOAM" folder then recreates it. See "mapOptimization" for implementation
```



### Use GPS：

#### woring

## Attention:

1.FAST-LIO2中对pose姿态是使用so3表示，而gtsam中，输入的relative_pose姿态是Euler RPY形式表示，需要使用罗德里格斯的公式进行转换更新。

2.参考yanliang-wang [FAST-LIO-LC](https://github.com/yanliang-wang/FAST_LIO_LC)中的iktree  reconstruct 



3.在walking数据集中，因为有个别数据是在同一个地方不断手持旋转激光雷达，旋转激光雷达的角度达到了保存关键帧的阈值，在短时间内，保存了多帧相似的关键帧，导致ISAM2出现特征退化，进而里程计跑飞，可以根据数据集的情况适当调整关键帧选取的阈值参数。



## Acknowledgements 

​	In this project, the LIO module refers to [FAST-LIO](https://github.com/hku-mars/FAST_LIO) and the pose graph optimization refers to [FAST_LIO_SLAM](https://github.com/gisbi-kim/FAST_LIO_SLAM) and [LIO_SAM](https://github.com/TixiaoShan/LIO-SAM).The mainly idea is for [FAST_LIO_LC](https://github.com/yanliang-wang/FAST_LIO_LC).Thanks there great work .

​	Also thanks yanliang-wang、minzhao-zhu、peili-ma  's  great help .

​																																														edited by kaho 2022.616