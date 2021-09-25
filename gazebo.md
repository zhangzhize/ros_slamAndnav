## 如何使用GAZEBO中的插件
  在gazebo中加入一些插件可以用来**仿真机器人的传感器、执行器的特性**,方法是通过```<gazebo>```元素中的```<plugin>```标签描述
```	

<gazebo reference="your_link_name">
	<plugin name = "your_link_laser_controller" filename="libgazebo_ros_laser.so">
		...plugin parameters
	</plugin>
</gazebo>
```
   同时gazebo默认支持很多常用设备，可以在 /usr/lib/x86_64-linux-gnu/gazebo-9/plugins 或 /opt/ros/melodic/lib 文件目录中找到相应的插件
## ros_control
* ros_control是一个**功能包**，是通过重写ros原有的pr2_mechanism包，进行推广，使得可以控制所有机器人
* ros_control以执行器**关节状态数据**和**期望**为输入，使用一般的反馈控制机制（通常PID），来进行控制输出（通常为力矩）
* ros_controllers:包含有可用的插件列表。主要实现**PID控制**，完成速度、位置等闭环控制，**指令发至gazebo或实体机器人**。可以自己编写代码创建控制插件。
	* effort_controllers:通过一个**期望扭矩/力矩**控制关节，包含的接口有：
		* joint_effort_controller
		* joint_position_controller
		* joint_velocity_controller

	* joint_state_controller:读取所有关节位置
	* position_controllers:一次设置一个或多个关节位置
		* joint_position_controller
		* joint_group_position_controller

	* velocity_controllers:一次设置一个或多个关节速度
		* joint_velocity_controller
		* joint_group_velocity_controller

	* joint_trajectory_controllers:用于为整个轨迹加附加功能
		* position_controller
		* velocity_controller
		* effort_controller
		* position_velocity_controller
		* position_velocity_acceleration_controller

* 使用ros_controller
	* 1.在urdf中添加```<transmission>```元素，eg：
```
<transmission name="simple_trans">
	<type>transmission_interface/SimpleTransmission</type>
	<joint name="foo_joint">
		<hardwareInterface>EffortJointInterface</hardwareInterface>
	</joint>
	<actuator name="foo_motor">
		<mechanicalReduction>50</mechanicalReduction>
		<hardwareInterface>EffortJointInterface</hardwareInterface>
	</actuator>
</transmission>
```
		* ```<joint name="">```:必须为对应urdf中定义的关节名称
		* ```<type>```:传输类型。该插件当前仅实现了“transmission_interface/SimpleTransmission",不要更改
		* ```<hardwareInterface>```:在```<actuator>```和```<joint>```标签中，即要加载的硬件接口（位置，速度或力矩接口）。当前仅实现了EffortJointInterface这一功能，不要更改
	* 2.添加gazebo_ros_control插件
```
<gazebo>
	<plugin name="gazebo_ros_control" filename="libgazebo_ros_control.so">
		<robotNamespace>/MYROBOT</robotNamespace>
	</plugin>
<gazebo>
```

		* ```<robotNamespace>```:用于当前插件实例化的ROS命名空间，默认为urdf/sdf中机器人的名称
		* ```<controlPeriod>```:控制器的更新周期(单位秒)，默认为gazebo的周期
		* ```<robotParam>```:urdf文件在参数服务器上的位置，默认为‘/robot_description’
		* ```<robotSimType>```:自定义的机器人仿真接口的pluginlib名称，默认为'DefaultRobotHWsim'
* 创建.yaml配置文件
	* PID系数和控制器配置必须保存在yaml文件中，通过launch文件加载到参数服务器中。
```
rrbot:
	joint_state_controller:
		type:joint_state_controller/JointStateController
		publish_rate:50
	joint_state_controller:
		type:effort_controllers/JointPositionController
		joint:joint1
		pid:{p:100,i:0.01,d:10.0}
```

	* 创建launch文件
```
<launch>

  <!-- Load joint controller configurations from YAML file to parameter server -->
  <rosparam file="$(find rrbot_control)/config/rrbot_control.yaml" command="load"/>

  <!-- load the controllers -->
  <node name="controller_spawner" pkg="controller_manager" type="spawner" respawn="false"
    output="screen" ns="/rrbot" args="joint1_position_controller joint2_position_controller joint_state_controller"/>

  <!-- convert joint states to TF transforms for rviz, etc -->
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"
    respawn="false" output="screen">
    <remap from="/joint_states" to="/rrbot/joint_states" />
  </node>

</launch>

```

	* 使用rostopic pub -1...  发送命令，或者使用rqt发送
* urdf文件详解
	* [见csdn收藏夹](https://blog.csdn.net/qq_16775293/article/details/88379988)
	* joint元素的type标签
	* revelote  可以绕着一个轴旋转的**铰链关节，有最大值和最小值限制**。
	* continuous 连续的铰链关节，可以绕一个轴旋转，**没有最大值和最小值限制**
	* prismatic 滑动关节，可以沿着一个轴滑动，没有最大值和最小值限制
	* fixed 不实际的关节，无法运动，自由度被锁定。这种类型的关节不需要指定轴、动力学特征、标度和最大值最小值限制
	* floating 6自由度的关节
	* planar 该关节在一个平面内运动，垂线是运动轴
* unitree_a1仿真模型的一些参数
	* hip关节的扭矩、速度和位置限制
		* max torque : 33.5
		* max velocity : 21
		* 位置：[-46,46],(单位：度)
	* thigh关节的扭矩、速度和位置限制
		* max torque : 33.5
		* max velocity : 21
		* 位置：[-60,240],(单位：度)
	* calf关节的扭矩、速度和位置限制
		* max torque : 33.5
		* max velocity : 21
		* 位置：[-154.5,-52.5],(单位：度)
	* 部件的重量
		* trunk: 6.0
		* hip: 0.696
		* thigh: 1.013
		* calf: 0.166
	* 其余参数见a1_description/urdf/const.xacro
