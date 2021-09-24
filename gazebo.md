## 如何使用GAZEBO中的插件
  在gazebo中加入一些插件可以用来**仿真机器人的传感器、执行器的特性**,方法是通过```<gazebo>```元素中的```<plugin>```标签描述
```	

<gazebo reference="your_link_name">
	<plugin name = "your_link_laser_controller" filename="libgazebo_ros_laser.so">
		...plugin parameters
	</plugin>
</gazebo>
```
   同时gazebo默认支持很多常用设备，可以在 /usr/lib/x86_64-linux-gnu/gazebo-9/plugins 或 /opt/ros/melodic/lib 文件目录中找到

