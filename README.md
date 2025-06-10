# ros1_SourceCodeCompilation
ros1 源码编译，已经在ubuntu22上编译通过
=============**主要介绍ros1的源码编译**============
##文档文件内容
#rosconsole-concise_output_roso.zip    升级之后的rosconsole，rosconsole文件是修改且已将验证过可以编译的
#noetic-ros_comm.rosinstall            ros.install文件，按下列步骤拉取不成功，可以直接引用，当然这是comm版本的
#src                                   comm版本的ros源码
#noetic_Installation_Source - ROS Wiki.pdf    官方源码编译安装文档
===**参考链接**===
##1.参考链接
ubunt22.03源码编译ros 1 noetic：     https://blog.csdn.net/u010085528/article/details/146412417
官网源码安装文档：   http://wiki.ros.org/noetic/Installation/Source


===**安装步骤**===
安装步骤
##1.安装依赖
```
	$ sudo apt-get install python3-rosdep python3-rosinstall-generator python3-vcstools python3-vcstool build-essential
```
##2.初始化rosdep
```
	$ sudo rosdep init
	$ rosdep update
```
##3.创建工作空间
```
	$ mkdir ~/ros_catkin_ws
	$ cd ~/ros_catkin_ws
```
##4.拉一个库文件，这个地方或许会下载不成功，可以引用本文件里面的noetic-ros_comm.rosinstall，也可在其他机器上下载放进去
```
	$ rosinstall_generator desktop --rosdistro noetic --deps --tar > noetic-desktop.rosinstall       
```
##5.拉源码和依赖
```	
	$ mkdir ./src
	$ vcs import --input noetic-desktop.rosinstall ./src
	$ rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro noetic -y
```
	
##6.编译，编译可能会有很多失败的地方，可以选择跳过不必要的包，当编译失败的时候可以用--from-pkg  packagename从这个地方开始往下编译，主要遇到的问题是rosconsole编译不
##成功，可以参考后面的常见问题以及解答，我这边已经整理好完整的rosconsole，可以直接替换引用，当然具体问题具体分析。
```
	$ ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release \
		-DPYTHON_EXECUTABLE=/usr/bin/python3  \
		--from-pkg rosconsole   \
		-Dlog4cxx_DIR=/usr/include/log4xx/
```

##7.引用环境变量并且验证测试,要想改变环境路径可以参考官方文档
```
	$ source ~/ros_catkin_ws/install_isolated/setup.bash
	$ roscore
```
===**常见问题以及解答**===
#1.指定某一些安装包
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release \
  --pkg catkin roscpp rospy rosgraph rosconsole roslib rostopic roslaunch rosparam \
  std_msgs sensor_msgs message_generation message_runtime  -DPYTHON_EXECUTABLE=/usr/bin/python3
  
#2.跳过某一些安装包
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release \
  --ignore-pkg rviz gazebo_ros navigation move_base map_server rqt* tf2_ros tf2_tools \
  robot_state_publisher joint_state_publisher image_transport  -DPYTHON_EXECUTABLE=/usr/bin/python3
  
#3.从某一个失败的开始安装
--from-pkg  packagename

#4.Log4cxx rosconsole编译不过问题 (以下是一些解决思路)，最终生成的rosconsole在本文件夹下，放进去直接编译就行
	版本更新后导致的问题

	rosconsole使用了Log4cxx, Fedora 38 官方源的 0.13.0 Log4cxx 在ROS安装编译时，由于会引发错误，Github repo上已经有commit修复了这个问题

	笔者解决方法

	    在src文件夹下删去vcstools下载的rosconsole整个文件夹
	    从此github repo (https://github.com/lucasw/rosconsole) 克隆branch "concise_output_roso" 放于src文件夹中
	    修改rosconsole/src/rosconsole/impl/rosconsole_log4cxx.cpp 文件中以下行，为以下所示 (参考：https://github.com/ros/rosconsole/pull/54， https://github.com/ros/rosconsole/commit/		e3753eec58bf4e76012d019fd307349f94d1d0be）

	// line 189
	 logger->forcedLog(g_level_lookup[level], str, log4cxx::spi::LocationInfo(file, log4cxx::spi::LocationInfo::calcShortFileName(file), function, line));

	// line 221
	 log4cxx::spi::LoggerRepositoryPtr repo = static_cast<log4cxx::spi::LoggerRepositoryPtr>(log4cxx::Logger::getLogger(ROSCONSOLE_ROOT_LOGGER_NAME)->getLoggerRepository());

	// line 388
	 log4cxx::Logger::getRootLogger()->getLoggerRepository()->shutdown();

#5.Log4cxx 系统库调用的问题

	(参考 [ros-noetic-mavros] error: ‘shared_mutex’ in namespace ‘std’ does not name a type · Issue #148 · acxz/pkgbuilds)

	更改系统Log4cxx库的头文件/usr/include/log4cxx/boost-std-configuration.h 从

	#define STD_SHARED_MUTEX_FOUND 1
	#define Boost_SHARED_MUTEX_FOUND 0

	改成

	#define STD_SHARED_MUTEX_FOUND 0
	#define Boost_SHARED_MUTEX_FOUND 1
