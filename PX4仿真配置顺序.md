### PX4仿真配置顺序

#### 下载固件

- 推荐ssh git clone 的方式，速度更快

``` shell
git clone --branch v1.12.3 git@github.com:PX4/PX4-Autopilot.git
```

#### 编译运行

```shell
cd /path/to/PX4-Autopilot
make px4_sitl gazebo 
```

- 编译过程中会下载一些东西 ，失败了多尝试几次。

#### 环境设置

https://docs.px4.io/v1.12/en/simulation/ros_interface.html该链接下有完整的配置

- mavros下模型
- octomap gazebo环境中快速建立地图 后续可用
- mavros将机载状态切换到offboard模式的步骤

```shell
cd <PX4-Autopilot_clone>
DONT_RUN=1 make px4_sitl_default gazebo
source ~/catkin_ws/devel/setup.bash    # (optional)
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
roslaunch px4 mavros_posix_sitl.launch
```

#### gazebo插件设置

- 原始的场景中使用/mavros/local/odom 大概是这个话题的odom信息不准

采用libgazebo_ros 中的p3d插件来获得世界坐标系下的准确odom信息

1. 需要设置当前环境中的ros pluigin 的环境里面是否有相应的p3d插件所在目录 开始大概率没有
   设置方法：

   ```shell
   export GAZEBO_PLUGIN_PATH=$GAZEBO_PLUGIN_PATH:${BUILD_DIR}/build_gazebo:/usr/lib/x86_64-linux-gnu/gazebo-11/plugins
   ```

   事先先看看是否有gazebo-11/plugins ，若没有sudo apt install ros-noetic-gazebo 之类的，具体可询问gpt

2. 在/home/xep/PX4-Autopilot/Tools/sitl_gazebo/models/iris/iris.sdf文件中添加下列插件：
   ```xml
   <!-- Fake localization plugin -->
   <!-- 真实odom插件 -->
       <plugin name="ground_truth_odometry" filename="libgazebo_ros_p3d.so">
         <alwaysOn>true</alwaysOn>
         <updateRate>100.0</updateRate>
         <bodyName>base_link</bodyName>
         <topicName>base_pose_ground_truth</topicName>
         <gaussianNoise>0.01</gaussianNoise>
         <frameName>world</frameName>
           <!-- initialize odometry for fake localization-->
         <xyzOffsets>0 0 0</xyzOffsets>
         <rpyOffsets>0 0 0</rpyOffsets>
       </plugin>
   ```

#### 测试环境

再次运行roslaunch px4 mavros_posix_sitl.launch ， rostopic list中查询是否有base_pose_ground_truth这个话题

#### PX4ctrl测试接口并且脚本文件起飞

在run_ctrl.launch文件中将odom话题改为base_pose_ground_truth相应话题，随后启动，再自动起飞脚本。

