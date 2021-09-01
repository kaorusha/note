# ROS1 and PX4 
## Setup `PX4` <---> `mavros` <---> `QGC` communication

When `mavros` runs on remote PC, the communication with `PX4` is through `telemetry` or a [wifi module](https://docs.px4.io/master/en/telemetry/esp8266_wifi_module.html). The key is to turnoff the auto connection of `QGC` and `PX4` via `telemetry`(see the [instructions](https://github.com/mavlink/mavros/issues/624)) and this [answer](https://github.com/mavlink/mavros/issues/878). Or the `mavros` is not able to use the `telemetry` to comminicate with `PX4`. The `QGC` also runs on remote PC and can communicate with `mavros` with UDP. The result is like `FCU <---telemetry---> MAVROS <---UDP---> QGC`.
> Note: after changing `QGC` configuration, the `telemetry` won't auto connect to `PX4` and so the red LED won't blink like it does when `FCU <---tellemetry---> QGC` and will start blinking until `mavros` was launched.

When `mavros` runs on a companion computer like single board computer mounted on the drone, the default `fcu_url:=/dev/ttyACM0:57600` means the companion computer connect with fcu via usb port. If `QGC` run on remote PC, SSH to companion computer and launch `mavros` use this [command](https://blog.csdn.net/qq_38649880/article/details/88342904) and use the IP of the remote PC `gcs_url:=udp://@xxx.xxx.xxx.xxx` or search udp automatically: `gcs_url:=udp-b://@`.

## Use /mavros/vision/pose to do auto takeoff and navigation
```sh
# shell A on remote PC
source /opt/ros/noetic/setup.bash
source catkin_ws/devel/setup.bash
roscore
```
```sh
# shell B (on companion computer with lidar)
source /opt/ros/noetic/setup.bash
source catkin_ws/devel/setup.bash
roslaunch ldlidar LD06.launch
```
```sh
# shell C on remote PC
cd PX4-Autopilot/
source /opt/ros/noetic/setup.bash
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
roslaunch px4 bringup.launch
```
```sh
# shell D on remote PC
cd <hector slam launch file directory>
source /opt/ros/noetic/setup.bash
roslaunch hector_slam_xtdrone.launch
```
```sh
# shell E on remote PC
cd <laser slam script file directory>
source /opt/ros/noetic/setup.bash
python3 vehicle_visual_odom_pub.py iris_rplidar 0 2d /poseupdate
```
