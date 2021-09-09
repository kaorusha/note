# ROS1 and PX4 
## Setup `PX4` <---> `mavros` <---> `QGC` communication
### with telemetry
When `mavros` runs on remote PC, the communication with `PX4` is through `telemetry` or a [wifi module](https://docs.px4.io/master/en/telemetry/esp8266_wifi_module.html). The key is to turnoff the auto connection of `QGC` and `PX4` via `telemetry`(see the [instructions](https://github.com/mavlink/mavros/issues/624)) and this [answer](https://github.com/mavlink/mavros/issues/878). Or the `mavros` is not able to use the `telemetry` to comminicate with `PX4`. The `QGC` also runs on remote PC and can communicate with `mavros` with UDP. The result is like `FCU <---telemetry---> MAVROS <---UDP---> QGC`.
> Note: after changing `QGC` configuration, the `telemetry` won't auto connect to `PX4` and so the red LED won't blink like it does when `FCU <---tellemetry---> QGC` and will start blinking until `mavros` was launched.

When connectting `FCU <---telemetry---> MAVROS` and `FCU <---telemetry---> QGC`the distance_sensor publish rate is 0.3 hz while `FCU <---usb---> MAVROS` the rate is about 10 hz, which is as fast as `FCU <---usb---> QGC`. The suggest [solution](https://github.com/mavlink/mavros/issues/1182) is changing `MAV_0_RATE` but this does not help with `distance_sensor`. And the [Holybro](http://www.holybro.com/manual/Telemetry-Radio-V3-Quick-Start-Guide.pdf) telemetry module does not support 115200 baud rate. The [default setting](https://docs.px4.io/master/en/peripherals/mavlink_peripherals.html#mavlink-instances) will generally be acceptable, but might be reduced if the telemetry link becomes saturated and too many messages are being dropped. (Reduce the meassge might be helpful by changing `MAV_X_MODE` other than `Normal`.)

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
### RTT too High For Timesync warning for real drone
> Note: 
Edit `px4_config.yaml` set the `timesync_rate` from origin 10(suggested) to 0.1 as this [link](https://discuss.ardupilot.org/t/rtt-too-high-for-timesync-with-sitl-mavros/38224/6)
### with UDP
#### `mavlink-router` [recommand](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html#companion-computer-setup)
##### Companion computer set up
The [TELEM2](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html#hardware-setup) is the default port for connecting pixhawk to companion computer by a uart-to-usb convertor like [FTDI](https://docs.px4.io/master/en/peripherals/companion_computer_peripherals.html#ftdi-devices) and [other FTDI devices](https://www.ruten.com.tw/item/show?21950031316494). We can also link directly through the usb port from the side of pixhawk.
##### Install mavlink router
The [repo](https://github.com/mavlink-router/mavlink-router) and the [config file example](http://bellergy.com/6-install-and-setup-mavlink-router/).
Get the following warning message:
```sh
# Mavlink-router can send both to mavros by 14540 and to GCS by 14550
$ mavlink-routerd -e 172.16.173.203:14540 -e 172.16.173.203:14550
Open UDP [4] 172.16.173.203:14550  
Error while trying to write serial port latency: Unknown error -1
Open UART [5] /dev/ttyACM0 *
UART [5] speed = 115200
Open TCP [6] 0.0.0.0:5760 *
22 messages to unknown endpoints in the last 5 seconds
```
The serial port latency [issue](https://github.com/mavlink-router/mavlink-router/issues/213) related to `TIOCSSERIAL` may not be supported by newer kernels, but it should continue to work regardless of that.
The “25 messages to unknown endpoints in the last 5 seconds” warning disappeared after running `mavros` on the remote PC. 
> Note: when using `PX4 <--- usb ---> rpi 0 w <--- mavlink-router through wifi(UDP) ---> QGC on PC`, the usb cable from `Pixhawk` to `rpi 0 w` should use the cable that can be recognized by QGC, not the cable which is charging only.

> `Rostopic --wall-time` will give the accurate publish rate. But if `mavros` were killed then re-launched, the rate will automatically switched to sim-time. Need re-run the command.

> Note: The [VPN](https://docs.px4.io/master/en/peripherals/companion_computer_peripherals.html#data-telephony-lte) on both `GCS` computer and companion computer can help to do secure data encryption and use static IP address and so the `mavlink-router` config does not need to change over time. 

#### `mavros` runs on companion computer and GCS on another computer
##### Install mavros on companion computer
[steps](https://junmo1215.github.io/tutorial/2019/07/14/tutorial-install-ROS-and-mavros-in-raspberry-pi.html)
When `mavros` runs on a companion computer like single board computer mounted on the drone, the default `fcu_url:=/dev/ttyACM0:57600` means the companion computer connect with fcu via usb port to the `TELEM2` port of the Pixhawk4 follow the [wiring](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html) .

If `QGC` run on remote PC, SSH to companion computer and launch `mavros` use this [command](https://blog.csdn.net/qq_38649880/article/details/88342904) and use the IP of the remote PC `gcs_url:=udp://@xxx.xxx.xxx.xxx` or search udp automatically: `gcs_url:=udp-b://@`.

## Use /mavros/vision/pose to do auto takeoff and navigation
Check and export `ROS_MASTER_URI`
```sh
# shell A on remote PC
source /opt/ros/noetic/setup.bash
source catkin_ws/devel/setup.bash
roscore
```
Check and export `ROS_MASTER_URI` and `ROS_HOSTNAME` on the companion computer
```sh
# shell B (on companion computer with lidar)
source /opt/ros/noetic/setup.bash
source catkin_ws/devel/setup.bash
roslaunch ldlidar LD06.launch
# of using uart GPIO port, using the following command (the lidar driver is from our modified branch)
roslaunch ldlidar LD06.launch device:=/dev/ttyS0
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
# shell D (on companion computer with lidar)
mavlink-routerd -e <remote PC IP>:14540 -e <remote PC IP>:14550
```
open QGC and to the Mavlink shell
```sh
nsh> vl53l1x start -X
```
```sh
# shell E on remote PC
cd <laser slam script file directory>
source /opt/ros/noetic/setup.bash
python3 vehicle_visual_odom_pub.py iris_rplidar 0 2d /poseupdate
```
```sh
# shell F on remote PC
cd <hector slam launch file directory>
source /opt/ros/noetic/setup.bash
roslaunch hector_slam_xtdrone.launch
```