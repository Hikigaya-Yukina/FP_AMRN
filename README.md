# Autonomous Mobile Robot Mapping & Navigation in Virtual Factory

![Ubuntu 20.04](https://img.shields.io/badge/OS-Ubuntu_20.04-informational?style=flat&logo=ubuntu&logoColor=white&color=2bbc8a)
![ROS Noetic](https://img.shields.io/badge/Tools-ROS_Noetic-informational?style=flat&logo=ROS&logoColor=white&color=2bbc8a)
![C++](https://img.shields.io/badge/Code-C++-informational?style=flat&logo=c%2B%2B&logoColor=white&color=2bbc8a)
![Python](https://img.shields.io/badge/Code-Python-informational?style=flat&logo=Python&logoColor=white&color=2bbc8a)

This is the final project report for the **ME5413 Autonomous Mobile Robotics course at the National University of Singapore**, completed by Group 6.

**Authors:**   chen Guanyu, Wu Zhuojun, Xu Boyang, Hao Yuqi, Miao Chenhui, Zhou Yinhong


<img src="src/me5413_world/media/gazebo_world.jpg" width="450" alt="virtual_factory"/><img src="report/images/map_info.png" width="350" alt="map_info"/>

<h2 id="1"> Installation</h2>

This repo is a ros workspace, containing three rospkgs:
* `interactive_tools` are customized tools to interact with gazebo and your robot
* `jackal_description` contains the modified jackal robot model descriptions
* `me5413_world` the main pkg containing the gazebo world, and the launch files

**Note:** If you are working on this project, it is encouraged to fork this repository and work on your own fork!

After forking this repo to your own github:

```bash
# Clone your own fork of this repo (assuming home here `~/`)
cd
git clone https://github.com/<YOUR_GITHUB_USERNAME>/ME5413_Final_Project.git
cd ME5413_Final_Project

# Install all dependencies
rosdep install --from-paths src --ignore-src -r -y

# Build
catkin_make
# Source
source devel/setup.bash
```

To properly load the gazebo world, you will need to have the necessary model files in the `~/.gazebo/models/` directory.

<h2 id="1"> 1. Project Decription</h2>

In this mini-factory environment:
* 3 target areas
* 1 restricted area

The aim of the project is to design a robot navigation software stack that can:
* From the starting point, move to the given pose within each area in sequence:
  * Assembly Line 1, 2
  * Random Box 1, 2, 3, 4
  * Delivery Vehicle 1, 2, 3

Task 1 Mapping

In this task, we manually map the environment using SLAM.
* Maps Building: use the SLAM alogrithms to map the environment
  * SLAM Point LIO: for its accuracy 
  * SLAM ALOAM & FLOAM: for the modernity 
* Maps Generating: caputure 3D point cloud maps through SLAM and convert them to 2D maps
  * 3D point cloud maps saving
  * Filter the useless points and adjust the orientations of PCDs
  * Convert 3D maps to 2D maps
 
Task 2 Navigation



**Files included:**
* Report: Group6_FinalProject.pdf
* PPT: Presentation.pptx
* Video Recording:
  * Mapping: one **fastlio.mp4** file available in: [Google Drive Link](https://drive.google.com/file/d/1EEQBYImnJs4QajZ1OG1U6qmV-J9dTWL5/view?usp=drive_link)
  * Navigation: one **Video_Uncompressed.mp4** file available in: [Google Drive Link](https://drive.google.com/file/d/1lj-83mchuxrjdUUB504Vcz-bM367FJpr/view?usp=share_link)
* Map file:
  1) one 2D projection map file **map.pgm**
  2) one 3D pointcloud mapping file **map_pointcloud.pcd** available in :[Google Drive Link](https://drive.google.com/file/d/19SF8SPli2iWed0avGKpOY63t1eXppWkJ/view?usp=share_link)

<h2 id="2"> 2. Prerequisites</h2>

### 2.1 **Ubuntu** and **ROS**
* Ubuntu >= 20.04
* ROS    >= Melodic. [ROS Installation](http://wiki.ros.org/ROS/Installation)

### 2.2 **PCL && Eigen**
* PCL    >= 1.8,   Follow [PCL Installation](http://www.pointclouds.org/downloads/linux.html).
* Eigen  >= 3.3.4, Follow [Eigen Installation](http://eigen.tuxfamily.org/index.php?title=Main_Page).

### 2.3 **livox_ros_driver**
* Follow [livox_ros_driver Installation](https://github.com/Livox-SDK/livox_ros_driver).

*Remarks:*
- **livox_ros_driver** must be installed and **sourced** as FAST-LIO must support Livox serials LiDAR firstly.
- How to source? Add the line `source $Licox_ros_driver_dir$/devel/setup.bash` to the end of file `~/.bashrc`, where `$Licox_ros_driver_dir$` is the directory of the livox ros driver workspace (should be the `ws_livox` directory if you completely followed the livox official document).

### 2.4 robot-pose-ekf
```
sudo apt-get install ros-noetic-robot-pose-ekf
```

### 2.5 navigation global_planner
```
sudo apt-get install ros-noetic-navigation
```

### 2.6 folder to be created
create a folder named `include` in`src/aster_ws/src/Astar_planner`. And an empty folder named `astar_planner` in it.

### Task 1 Mapping
After building the whole files, in the folder starting Point_LIO:
```bash
# Launch Gazebo World together with our robot
roslaunch me5413_world world.launch

# In another terminal
roslaunch me5413_world Point_lio.launch
```
Once you mapped the whole world you can find the **PCD** files in ~/src/Point_LIO/PCD.

If you want to use ALOAM to get a 3D map:
```bash
# Launch Gazebo World together with our robot
source devel/setup.bash
roslaunch me5413_world world.launch

# In another terminal
source devel/setup.bash
roslaunch me5413_world aloam.launch

# Get the rosbags of the map (you can start a new terminal to do so)
# This performance can be done after you maped the whole world but the terminal does not be closed
rosbag record /laser_cloud_map -O map3
rosrun pcl_ros bag_to_pcd map3.bag /laser_cloud_map map3.pcd
```

You may process the useless point in the 3D PCD file and convert the map to a 2D map:
```bash
# Use pass through filer and Radius Outlier Removal
# Put the PCD maps in the home directory
source devel/setup.bash
roslaunch pcd2pgm run.launch

rosrun map_server map_saver

#If the pcd map's orientations are not good, you can rotate it first (This script also has the function to filter the point)
# Put the PCD maps in the home directory
source devel/setup.bash
roslaunch read_pcd read_pcd.launch 
```
If you want to use EVO to evaluate the performance:
```bash
# Record the datas in the beginning in a new terminal
rosbag record /gazebo/ground_truth/state /odometry/filtered -O Truth_Odom.bag

# After that:
evo_ape bag Truth_Odom.bag /gazebo/ground_truth/state /odometry/filtered -r full -va --plot --plot_mode xy
```
### Task 2 Navigation

After completing the mapping, end the mapping process and restart gazebo. To load the navigation stack, execute the following command in a new terminal window.
```
source devel/setup.bash
roslaunch me5413_world world.launch

source devel/setup.bash
roslaunch me5413_world navigation.launch
```
Select the relevant topic under global path, and then press the button on simplePanel to set the desired goal pose.

<img src="report/images/navigation_result.png" width="500" alt="naviagation_result_image"/>
