# m-explore ROS2 port

ROS2 package port for multi robot exploration of [m-explore](https://github.com/hrnr/m-explore). Currently tested on Eloquent, Dashing, Foxy, and Galactic distros.

## Autonomous exploration

### TB3
https://user-images.githubusercontent.com/8033598/128805356-be90a880-16c6-4fc9-8f54-e3302873dc8c.mp4


### On a JetBot with realsense cameras
https://user-images.githubusercontent.com/18732666/128493567-6841dde0-2250-4d81-9bcb-8b216e0fb34d.mp4


Installing
----------

No binaries yet.

Building
--------

Build as a standard colcon package. There are no special dependencies needed
(use rosdep to resolve dependencies in ROS). 

RUNNING
-------
To run with a params file just run it with
```
ros2 run explore_lite explore --ros-args --params-file <path_to_ros_ws>/m-explore/explore/config/params.yaml
```

### Running the demo with TB3
Install nav2 and tb3 simulation. You can follow the [tutorial](https://navigation.ros.org/getting_started/index.html#installation).

Then just run the nav2 stack with slam:

```
export TURTLEBOT3_MODEL=waffle
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/opt/ros/${ROS_DISTRO}/share/turtlebot3_gazebo/models
ros2 launch nav2_bringup tb3_simulation_launch.py slam:=true
```

And run this package with
```
ros2 launch explore_lite explore.launch.py
```

You can open an rviz2 and add the exploration frontiers marker to see the algorithm working and choose a frontier to explore.

#### TB3 troubleshooting (with foxy)
If you have trouble with TB3 in simulation like we did, add this extra steps for configuring it.

```
source /opt/ros/${ROS_DISTRO}/setup.bash
export TURTLEBOT3_MODEL=waffle
sudo rm -rf /opt/ros/${ROS_DISTRO}/share/turtlebot3_simulations
sudo git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations /opt/ros/${ROS_DISTRO}/share/turtlebot3_simulations
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/opt/ros/${ROS_DISTRO}/share/turtlebot3_simulations/turtlebot3_gazebo/models
```

Then you'll be able to run it.

## Multirobot map merge

This package works with known and unknown initial poses of the robots. It merges the maps of the robots and publishes the merged map. Some results in simulation:

### Known initial poses (best results)
https://user-images.githubusercontent.com/8033598/144522712-c31fb4bb-bb5a-4859-b3e1-8ad665f80696.mp4

### Unknown initial poses 
It works better if the robots start very close (< 3 meters) to each other so their relative positions can be calculated properly.

https://user-images.githubusercontent.com/8033598/144522696-517d54fd-74d0-4c55-9aca-f1b9679afb3e.mp4

### ROS2 requirements

#### SLAM
Because of the logic that merges the maps, currently as a straight forward port to ROS2 from the ROS1 version, the SLAM needs to be done using the ROS1 defacto slam option which is [slam_gmapping](https://github.com/ros-perception/slam_gmapping), which hasn't ported officially to ROS2 yet. There is an unofficial port but it lacks to pass a namespace to its launch file. For that, this repo was tested with one of the authors of this package [fork](https://github.com/charlielito/slam_gmapping/tree/feature/namespace_launch). You'll need to git clone to your workspace and build it with colcon.


```
cd <your/ros2_ws/src>
git clone https://github.com/charlielito/slam_gmapping.git --branch feature/namespace_launch
cd ..
colcon build --symlink-install --packages-up-to slam_gmapping
```
#### Nav2 config files
This repo has some config examples and launch files for running this package with 2 TB3 robots and a world with nav2. Nonetheless, they are only compatible with the galactic branch and since some breaking changes were introduced in this branch, if you want to try it with another ros2 distro you'll need to tweak those param files for that nav2's distro version (which shouldn't be hard).

### Running the demo with TB3
First you'll need to launch the whole simulation stack, nav2 stacks and slam stacks per robot. For that just launch::
```
export TURTLEBOT3_MODEL=waffle
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/opt/ros/${ROS_DISTRO}/share/turtlebot3_gazebo/models
ros2 launch multirobot_map_merge multi_tb3_simulation_launch.py slam_gmapping:=True
```
Now run the merging node:
```
ros2 launch multirobot_map_merge map_merge.launch.py
```

By default the demo runs with known initial poses. You can change that by launching again both launch commands with with the flag `known_init_poses:=False`

Then you can start moving each robot with its corresponding rviz2 interface sending nav2 goals. To see the map merged just launch rviz2:
```
rviz2 -d <your/ros2_ws>/src/m-explore-ros2/map_merge/launch/map_merge.rviz
```

WIKI
----
No wiki yet.

COPYRIGHT
---------

Packages are licensed under BSD license. See respective files for details.
