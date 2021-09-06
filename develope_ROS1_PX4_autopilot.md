# ROS1 and PX4 
## Setup `PX4` <---> `mavros` <---> `QGC` communication

When `mavros` runs on remote PC, the communication with `PX4` is through `telemetry` or a [wifi module](https://docs.px4.io/master/en/telemetry/esp8266_wifi_module.html). The key is to turnoff the auto connection of `QGC` and `PX4` via `telemetry`(see the [instructions](https://github.com/mavlink/mavros/issues/624)) and this [answer](https://github.com/mavlink/mavros/issues/878). Or the `mavros` is not able to use the `telemetry` to comminicate with `PX4`. The `QGC` also runs on remote PC and can communicate with `mavros` with UDP. The result is like `FCU <---telemetry---> MAVROS <---UDP---> QGC`.
> Note: after changing `QGC` configuration, the `telemetry` won't auto connect to `PX4` and so the red LED won't blink like it does when `FCU <---tellemetry---> QGC` and will start blinking until `mavros` was launched.

When `mavros` runs on a companion computer like single board computer mounted on the drone, the default `fcu_url:=/dev/ttyACM0:57600` means the companion computer connect with fcu via usb port to the `TELEM 2` port of the Pixhawk4 follow the [wiring](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html) .
If `QGC` run on remote PC, SSH to companion computer and launch `mavros` use this [command](https://blog.csdn.net/qq_38649880/article/details/88342904) and use the IP of the remote PC `gcs_url:=udp://@xxx.xxx.xxx.xxx` or search udp automatically: `gcs_url:=udp-b://@`.

When connectting `FCU <---telemetry---> MAVROS` and `FCU <---telemetry---> QGC`the distance_sensor publish rate is 0.3 hz while `FCU <---usb---> MAVROS` the rate is about 10 hz, which is as fast as `FCU <---usb---> QGC`. The suggest [solution](https://github.com/mavlink/mavros/issues/1182) is changing `MAV_0_RATE` but this does not help with `distance_sensor`. And the [Holybro](http://www.holybro.com/manual/Telemetry-Radio-V3-Quick-Start-Guide.pdf) telemetry module does not support 115200 baud rate.
The usb and /dev/ttyACM0(when connecting with Pixhawk) is both default 9600 baud:
```sh
$ stty -F /dev/ttyUSB3
speed 9600 baud; line = 0;
-brkint -imaxbel
```
The baudrate of usb-telemetry changed after launching Mavros:
```sh
$ stty -F /dev/ttyUSB3
speed 57600 baud; line = 0;
min = 1; time = 0;
-brkint -icrnl -imaxbel
-opost
-isig -icanon -iexten -echo
```

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
open QGC and to the Mavlink shell
```sh
nsh> vl53l1x start -X
```
```sh
# shell D on remote PC
cd <laser slam script file directory>
source /opt/ros/noetic/setup.bash
python3 vehicle_visual_odom_pub.py iris_rplidar 0 2d /poseupdate
```
```sh
# shell E on remote PC
cd <hector slam launch file directory>
source /opt/ros/noetic/setup.bash
roslaunch hector_slam_xtdrone.launch
```
### RTT too High For Timesync warning for real drone
> Note: 
Edit `px4_config.yaml` set the `timesync_rate` from origin 10(suggested) to 0.1 as this [link](https://discuss.ardupilot.org/t/rtt-too-high-for-timesync-with-sitl-mavros/38224/6)

## Companion computer set up
### Install mavlink router (recommand)
The [repo](https://github.com/mavlink-router/mavlink-router) and the [config file example](http://bellergy.com/6-install-and-setup-mavlink-router/).
### Install mavros
[steps](https://junmo1215.github.io/tutorial/2019/07/14/tutorial-install-ROS-and-mavros-in-raspberry-pi.html)