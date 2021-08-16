# Setup and use ROS 2 with PX4
Compenion computer setup.
This document is the personal note as following [instructions](https://docs.px4.io/master/en/ros/ros2_comm.html#installation-setup) for doing setup. (The supporting OS is Ubuntu 18.04 or higher, so some additional package needed for **raspberry pi zero w** with **Raspbian** OS).

Px4 user document recommended [ROS2](https://docs.px4.io/master/en/ros/ros2.html) for better support:
>This contrasts with ROS (1), which communicates with PX4 via MAVROS/MAVLink, hiding PX4's internal architecture and many of its conventions (e.g. frame and unit conversions).
>ROS 2 (and the bridge) will become easier to use as the development team provide ROS 2 APIs to abstract PX4 conventions, along with examples demonstrating their use. These are planned in the near-term PX4 roadmap.

## Fast DDS library Installation from Sources
see [PX4 doc](https://docs.px4.io/master/en/ros/ros2_comm.html#install-fast-dds)
requirement:
* Fast RTPS(DDS) 2.0.0 (or later)
* Fast-RTPS-Gen 1.0.4 (not later!)
### Requirement
Follow the Fast DDS official [installation manual](https://fast-dds.docs.eprosima.com/en/latest/installation/sources/sources_linux.html)
* (on raspbian)Install gtest and gmock for raspbian:
```sh
sudo apt-get install libgtest-dev
sudo apt-get install libgmock-dev
```
### Use `Colcon` or `CMake`
Choose `Colcon` installation for `Raspbian`, and reboot after installing vcstools. If possible, use cross compiling because compile on Raspberry Pi zero w took really long time:
>Finished <<< fastrtps [4h 28min 9s]                                                            
Summary: 3 packages finished [4h 28min 35s]

`CMake` is suggested by PX4 for `Ubuntu` and will face the following error on `Raspbian`:</br>
>/usr/bin/ld: cannot find /usr/share/lintian/overrides: file format not recognized
>collect2: error: ld returned 1 exit status
>make[2]: *** [src/cpp/CMakeFiles/fastrtps.dir/build.make:2953: src/cpp/libfastrtps.so.2.0.0] Error 1
>make[1]: *** [CMakeFiles/Makefile2:1022: src/cpp/CMakeFiles/fastrtps.dir/all] Error 2
>make: *** [Makefile:163: all] Error 2
## Fast DDS-Gen installation
### Prerequisites
#### Java JDK 8
Required by `gradle`. Check by running:
```sh
$ java -version
java version "1.8.0_121"
```
[download](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) `Linux x64 Compressed Archive`
[install](https://docs.oracle.com/javase/8/docs/technotes/guides/install/linux_jdk.html#BJFJJEFG) or install globally by running:
```sh
sudo apt-get install openjdk-8-jdk
```
the above command solved the following error when building `Fast-RTPS-Gen`:
>FAILURE: Build failed with an exception.
>
>* What went wrong:
>Execution failed for task ':idl-parser:compileJava'.
> Could not find tools.jar. Please check that /usr/lib/jvm/java-8-openjdk-amd64 contains a valid JDK installation.
#### Gradle
For rasobian OS, install zip first:
```sh
sudo apt-get install zip unzip
``` 
### use v1.0.4
using this [command](https://docs.px4.io/master/en/dev_setup/fast-dds-installation.html#fast-rtps-gen)

## PX4_ROS_COM
Follow build [instruction](https://docs.px4.io/master/en/ros/ros2_comm.html#build-ros-2-workspace). If build failed, use a clean build should solve the error.
## Navigation 2
### lidar performance
From the [ROS Navigation Tuning Guide](https://kaiyuzheng.me/documents/navguide.pdf) the following lidar property will affect the localization performance.
#### Resolution
The map gride resolution is limited by the laser ranging and angle resolution. The costmap updating cost will be wasted if the gride is smaller than the laser scanning resolution.
#### Variance
Greater variance gives a higher spreading range of the particle filter that helps localization correctness. This variance value should follow the laser spec.
### Controller server configuration
Minimum x and y [velocity_threshold](https://navigation.ros.org/configuration/packages/configuring-controller-server.html) will affect the drifting sensibility of the drone.
### Behavior trees
[multiple goals](https://navigation.ros.org/behavior_trees/trees/nav_through_poses_recovery.html): 
behavior trees xml on branch [foxy-devel](https://github.com/ros-planning/navigation2/tree/foxy-devel/nav2_bt_navigator/behavior_trees) and on branch [main](https://github.com/ros-planning/navigation2/tree/main/nav2_bt_navigator/behavior_trees): the branch `foxy-devel` has no `navigate_through_poses_w_replanning_and_recovery.xml`.

## Odometry update
The real time MCU driver, on which the firmware responsible for motion controls published `/odom` topic for the navigation stack to subscribe. See example of [turtlebot3](https://github.com/ROBOTIS-GIT/OpenCR/blob/master/arduino/opencr_arduino/opencr/libraries/turtlebot3/examples/turtlebot3_burger/turtlebot3_core/turtlebot3_core.ino).
## PX4 Offboard Mode
need target setpoints > 2Hz
COM_OBL_ACT: Set offboard loss failsafe mode. now use 2: Return mode (but this mode requires GPS)
## ROS2 Launch System
The [document](https://design.ros2.org/articles/roslaunch.html) described the differences of launch system of ros and ros2.
### Parameters
No dynamic reconfigures package, instead, all parameter is local and dynamic reconfigureable, specified by the node.
### control the order and timing of launched nodes
### specify the events and responses of node life cycle conditions 

## DroneKit
This ardupilot use raspberry pi zero as a companion computer to deal with other computational heavy task like image processing.
[Instructions](https://ardupilot.org/dev/docs/raspberry-pi-via-mavlink.html#setup-the-rpi-software) to route the ground station to communicate with fly control unit through UART port with MAVLINK protocal. USB-wifi adapter to broaden the communication range limit.

## Simulation
The entry point for SITL is the shell script `PX4-AUTOPILOT/Tools/sitl_run.sh`. The environment variables such as world file and model file can be changed as [this](https://docs.px4.io/master/en/simulation/gazebo.html#loading-a-specific-world). Another file `PX4-AUTOPILOT/ROMFS/px4fmu_common/init.d-posix/rcS` set other environment variables that related to PX4, changing setting as this [instruction](https://docs.px4.io/master/en/simulation/#run-simulation-faster-than-realtime).
```sh
cd /PX4-AUTOPILOT/
export PX4_SITL_WORLD=$PWD/Tools/sitl_gazebo/worlds/indoor3.world 
# substitute other customized world file
echo $PX4_SITL_WORLD
# check the absolute directory is correct
make px4_sitl gazebo_iris_rplidar
# start simulation with previous specified world
```
print the following distance sensor messages:
```sh
pxh> listener distance_sensor
Instance 0:
 distance_sensor_s
        timestamp: 493360000  (0.020000 seconds ago)
        device_id: 10092548 (Type: 0x9A, SIMULATION:0 (0x00)) 
        min_distance: 0.2000
        max_distance: 15.0000
        current_distance: 0.2000
        variance: 0.0000
        h_fov: 0.0524
        v_fov: 0.0524
        q: [0.7071, -0.0000, -0.7071, -0.0000]  (Roll: 0.0 deg, Pitch: -90.0 deg, Yaw: -0.0 deg)
        signal_quality: 0
        type: 0
        orientation: 25

Instance 1:
 distance_sensor_s
        timestamp: 493364000  (0.016000 seconds ago)
        device_id: 10092804 (Type: 0x9A, SIMULATION:0 (0x01)) 
        min_distance: 0.2000
        max_distance: 15.0000
        current_distance: 2.5000
        variance: 0.0000
        h_fov: 0.0524
        v_fov: 0.0524
        q: [-0.7071, -0.0000, -0.7071, -0.0000]  (Roll: 0.0 deg, Pitch: 90.0 deg, Yaw: 0.0 deg)
        signal_quality: 66
        type: 0
        orientation: 24
```
### Read laser scan to ros topic
#### ros1
Gazebo must be launched with the ROS wrapper with [roslaunch](https://docs.px4.io/master/en/simulation/ros_interface.html#launching-gazebo-with-ros-wrappers) in order to get the data of Gazebo ROS laser plugin. 
After exporting `ROS_PACKAGE_PATH`, `px4` package name can be found by roslaunch. 
#### ros2
Change the laser plugin from ros1 plugin `gazebo_ros_laser` to ros2 plugin `gazebo_ros_ray_sensor` as this [example](https://github.com/ros-simulation/gazebo_ros_pkgs/wiki/ROS-2-Migration:-Ray-sensors).
## Add New Airframe
Modify 2 files at `PX4-Autopilot/ROMFS/px4fmu_common/init.d/airframes`: 
### PID gains and gyro cutoff frequency.
Take `4015_holybro_s500` as an example: [x500](https://docs.px4.io/master/en/frames_multicopter/holybro_x500_pixhawk4.html) and [s500v2](https://docs.px4.io/master/en/frames_multicopter/holybro_s500_v2_pixhawk4.html) use same configuration.
### Physical properties of the frame.
The following properties are specified for each propeller. See `4018_s500_ctrlalloc` as an example.  
* [Thrust Coeffieient](https://docs.px4.io/master/en/advanced_config/parameter_reference.html?#CA_MC_R0_CT) ([difinition](https://web.mit.edu/16.unified/www/FALL/thermodynamics/notes/node86.html#SECTION06374100000000000000))
* [Moment coefficient](https://docs.px4.io/master/en/advanced_config/parameter_reference.html?#CA_MC_R0_KM) ([definition](https://youtu.be/brDkdI1V0dA?t=368))

Duplicate these files to `init.d-posix` directory if also need to use them in SITL. 