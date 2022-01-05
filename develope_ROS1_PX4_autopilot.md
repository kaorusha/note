# ROS1 and PX4 
# Setup `PX4` <---> `mavros` <---> `QGC` communication
## with telemetry
When `mavros` runs on remote PC, the communication with `PX4` is through `telemetry` or a [wifi module](https://docs.px4.io/master/en/telemetry/esp8266_wifi_module.html). The key is to turnoff the auto connection of `QGC` and `PX4` via `telemetry`(see the [instructions](https://github.com/mavlink/mavros/issues/624)) and this [answer](https://github.com/mavlink/mavros/issues/878). Or the `mavros` is not able to use the `telemetry` to comminicate with `PX4`. The `QGC` also runs on remote PC and can communicate with `mavros` with UDP. The result is like `FCU <---telemetry---> MAVROS <---UDP---> QGC`.
> Note: after changing `QGC` configuration, the `telemetry` won't auto connect to `PX4` and so the red LED won't blink like it does when `FCU <---tellemetry---> QGC` and will start blinking until `mavros` was launched.

When connecting `FCU <---telemetry---> MAVROS` and `FCU <---telemetry---> QGC`the distance_sensor publish rate is 0.3 hz while `FCU <---usb---> MAVROS` the rate is about 10 hz, which is as fast as `FCU <---usb---> QGC`. The suggest [solution](https://github.com/mavlink/mavros/issues/1182) is changing `MAV_0_RATE` but this does not help with `distance_sensor`. And the [Holybro](http://www.holybro.com/manual/Telemetry-Radio-V3-Quick-Start-Guide.pdf) telemetry module does not support 115200 baud rate. The [default setting](https://docs.px4.io/master/en/peripherals/mavlink_peripherals.html#mavlink-instances) will generally be acceptable, but might be reduced if the telemetry link becomes saturated and too many messages are being dropped. (Reduce the message might be helpful by changing `MAV_X_MODE` other than `Normal`.)

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
## RTT too High For Timesync warning for real drone
> Note: 
Edit `px4_config.yaml` set the `timesync_rate` from origin 10(suggested) to 0.1hz as this [link](https://discuss.ardupilot.org/t/rtt-too-high-for-timesync-with-sitl-mavros/38224/6) for `telemetry radio` and 2hz for `mavlink router`.
## with UDP
### `mavlink-router` [recommand](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html#companion-computer-setup)
#### Companion computer set up
The [TELEM2](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html#hardware-setup) is the default port for connecting pixhawk to companion computer by a uart-to-usb convertor like [FTDI](https://docs.px4.io/master/en/peripherals/companion_computer_peripherals.html#ftdi-devices) and [other FTDI devices](https://www.ruten.com.tw/item/show?21950031316494). We can also link directly through the usb port from the side of pixhawk.
#### Install mavlink router
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

### `mavros` runs on companion computer and GCS on another computer
#### Install mavros on companion computer
[steps](https://junmo1215.github.io/tutorial/2019/07/14/tutorial-install-ROS-and-mavros-in-raspberry-pi.html)
When `mavros` runs on a companion computer like single board computer mounted on the drone, the default `fcu_url:=/dev/ttyACM0:57600` means the companion computer connect with fcu via usb port to the `TELEM2` port of the Pixhawk4 follow the [wiring](https://docs.px4.io/master/en/companion_computer/pixhawk_companion.html) .

If `QGC` run on remote PC, SSH to companion computer and launch `mavros` use this [command](https://blog.csdn.net/qq_38649880/article/details/88342904) and use the IP of the remote PC `gcs_url:=udp://@xxx.xxx.xxx.xxx` or search udp automatically: `gcs_url:=udp-b://@`.

# Use /mavros/vision/pose to do auto takeoff and navigation
> note: 
when debugging, Pixhawk 4 will start warning beap when not giving arm command after a long time. Disconnect the power input port to stop the warning beap while the USB port con still supply power for the board and other sensors.
## PX4 on real machine
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
source /opt/ros/melodic/setup.bash
source catkin_ws/devel/setup.bash
# default using uart GPIO port
roslaunch ldlidar LD06.launch
# of using CP2102 usb adapter, using the following command (the lidar driver is from our modified branch)
roslaunch ldlidar LD06.launch device:=""
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
### manual start sensor driver
open QGC and to the Mavlink shell
```sh
# start distance sensor driver
nsh> vl53l1x start -X -b 4
nsh> vl53l1x start -X -b 2 -R 24
```
> Note: Set [SENS_EN_VL53L1X](https://docs.px4.io/master/en/advanced_config/parameter_reference.html#SENS_EN_VL53L1X) to true to start all driver at startup with [default setting](https://docs.px4.io/master/en/modules/modules_driver_distance_sensor.html#vl53l1x).
### auto start sensor driver
When using multiple distance sensors, disable `SENS_EN_VL53L1X` and add extra startup [script](https://docs.px4.io/master/en/concept/system_startup.html#starting-additional-applications) to specify each sensor [port](http://www.holybro.com/manual/Pixhawk4-Pinouts.pdf) and [orientation](https://docs.px4.io/master/en/advanced_config/parameter_reference.html#SENS_CM8JL65_R_0) (defined as [MAV_SENSOR_ORIENTATION](https://mavlink.io/en/messages/common.html#DISTANCE_SENSOR)).
```sh
# read SD card of FMU on PC
cd <dir of sd card>
mkdir etc && cd etc
touch extras.txt
# and paste the script intended to run in the nsh
```
> Note: The files in the /fs/microsd/etc will exist after rebuilt firmware.
## PX4 Simulation with SITL gazebo
Replace the previous steps with the following [steps](http://docs.px4.io/master/en/simulation/ros_interface.html#launching-gazebo-with-ros-wrappers). Rebuild after changing `.sdf` file. ([troubleshooting](http://docs.px4.io/master/en/dev_setup/building_px4.html#general-build-errors))
```sh
# SITL
cd PX4-Autopilot/
source /opt/ros/noetic/setup.bash
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
roslaunch px4 indoor3.launch gui:=false
```
> Note: current lidar use `gazebo_lidar_plugin.cpp` that will return minimum distance if facing out of range area.
## Laser scan for localization on ROS
```sh
# shell E on remote PC
cd <XTDrone>/sensing/slam/laser_slam/script/
source /opt/ros/noetic/setup.bash
python3 vehicle_visual_odom_pub.py iris_rplidar 0 2d /poseupdate
```
> for simulation, remember to modify parameter `/use_sim_time` to true in the `hector_slam_xtdrone.launch` file.
```sh
# shell F on remote PC
cd <XTDrone>/sensing/slam/laser_slam/hector_slam/
source /opt/ros/noetic/setup.bash
roslaunch hector_slam_xtdrone.launch
```
### Auto takeoff
Check the local altitude and takeoff height is in local frame. 
```sh
# QGC mavshell(real machine) or SITL(simulation)
commander arm
commander takeoff
```
## Navigation
Connect `cmd_vel` to mavros `set_position_raw`
```sh
# shell G on remote PC
cd <XTDrone>/communication/
source /opt/ros/noetic/setup.bash
python3 multirotor_communication.py iris_rplidar 0 0
```
```sh
#shell H on remote PC
cd <XTDrone>/motion_planning/2d/
source /opt/ros/noetic/setup.bash
roslaunch launch/2d_motion_planning.launch # note the dir_name is motion_planning
```
# roslaunch on startup
## Get ROS_HOSTNAME automatically
The `~/.bashrc` file specifies `ROS_HOSTNAME` and `ROS_MASTER_URI`. These IP addresses must be set [explicitly](http://wiki.ros.org/ROS/NetworkSetup). However, the `hostname` can output raspberry pi's current IP, so we don't need to change `~/.bashrc` every time when IP changed, but the output IP add an space ant the end, so the "xxx.xxx.xxx.xxx" becomes "xxx.xxx.xxx.xxx ", and the IP can't be set correctly. To [trim the ending space](https://stackoverflow.com/questions/369758/how-to-trim-whitespace-from-a-bash-variable):
```sh
export ROS_HOSTNAME="$(echo -e "$(hostname -I)" | xargs)"
```  
### Troubleshooting
#### Given velocity for `setpoint_raw` with position masked and result in overshoot at the goal position and pullback repeatedly
Lower `MPC_XY_CRUISE` and `MC_YAWRATE_MAX` limit. The flight stack use `MPC_XY_CRUISE` as speed limit for offboard `PositionTarget`. And we can set the bitmask to select position or velocity control. When ignoring velocity setpoint, the flight stack will take care of speed control and slow down near goal. When ignoring position setpoint, the speed is calculated by offboard controller so the speed limit is crucial.
Higher the target publish rate for velocity setpoint to 100 hz, although 20 hz is good for position setpoint. And accelerate setpoint need evan higher rate (mentioned in this [blog](https://blog.csdn.net/benchuspx/article/details/115750466)).
> Note: `mavros` plugin `setpoint_position/velocity/accel/raw/trajectory` all call the `set_position_target_local_ned()` in file `setpoint_mixin.hpp` to send setpoint with type [SET_POSITION_TARGET_LOCAL_NED](https://mavlink.io/en/messages/common.html#SET_POSITION_TARGET_LOCAL_NED).
#### Given position for `setpoint_raw` with velocity masked but unable to set yaw
When given yaw, the drone attitude changed drastically and the estimator will lost its position. This might caused by not well-tunning yaw PID gain.
> According to this [blog](https://akshayk07.weebly.com/offboard-control-of-pixhawk.html). But yaw control could not be done correctly yet.

By lowering P `MC_YAW_P` to half as 1.4 ease the overshoot of yaw angle, so the vehicle yaw can be controlled by giving `sepoint_position` with yaw orientation. However when the vehicle reached the goal, it is very often that the yaw angle can not turned to the desired direction, and the local_planner eventually gives up. 
##### Workaround
Set yaw_rate of the setpoint_raw instead of controlling yaw angle directly, with yaw masked, yaw_rate can well and stably control the drone.
#### Position control: overshoot of offboard setpoint
Tune `MPC_XY_VEL_*` as [suggested](https://discuss.px4.io/t/pid-issue-when-firmware-update-from1-10-to-1-12/23385) and [this](https://discuss.px4.io/t/very-agressive-oscillation-in-position-mode/807/16)
#### lidar scanner will tilt and scan the body itself when simulation with gazebo 
This is caused by bending of the fixed link between lidar scanner and the body. To eliminate the bending of the link, use a lower moment of inertia of lidar scanner and a gentle acceleration in motion.
#### [ WARN] [1563590145.137041656]: Off Map 1.159504, -3.989272 
Set larger size for the local_cost_map. The message is telling the simulating step is outside the costmap.
#### [ERROR] [1634285684.713884704]: FCU: Failsafe enabled: No manual control stick input
Set `COM_RCL_EXCEPT` to 3 to allow arm, takeoff mission, and offboard mode.
#### Collision Prevention
For `position mode`, set `MPC_POS_MODE` to 3 or 0 to enable collision prevention, not the default 4 supporting [acceleration setpoint](https://discuss.px4.io/t/px4-vision-collision-prevention-feature-seems-not-to-work/23476/25). `CP_GO_NO_DATA` should be 1, not default 0 to prevent flying outside of [FOV](https://discuss.px4.io/t/px4-vision-collision-prevention-feature-seems-not-to-work/23476/32).
# Mission Mode
If the simulation reject to execute the mission, check the following params:
`EKF2_AID_MASK` including GPS, because this mode does not support local NED frame as the [doc](https://docs.px4.io/master/en/flight_modes/mission.html) said. And currently the PX4 and mavros vision estimate only support mavlink topic [VISION_POSITION_ESTIMATE](https://mavlink.io/en/messages/common.html#VISION_POSITION_ESTIMATE) in local coordinate.
`NAV_ACC_RAD` is adjusted with the takeoff height. The firmware code now requires height has to be larger than the radius by 1 meter, or shows the following error:
```sh 
Mission rejected: Takeoff altitude too low!
```
`COM_OBS_AVOID` set to 0 unless obstacle and trajectory message are publishing.
# Avoidance node
* [interface](https://docs.px4.io/master/en/computer_vision/path_planning_interface.html)
* [message flow](https://github.com/PX4/PX4-Avoidance#message-flows)

The local_planner use 4 thread, from line # 47 in `local_planner_nodelet.cpp` calls `onInit()`:
* main thread `startNode()` calls the main loop callback function
* thread for updating the waypoints and state from Mission Mode in line # 476 `setPlannerInfo()` as the input for the local planner
* thread for tf updating
* thread for dynamic parameter updating (implemented in `avoidance_node.cpp` line # 41 `AvoidanceNode::init()`)  
The main loop callback functions calls the following method to update the setpoint in line #283:
```cpp
void LocalPlannerNodelet::calculateWaypoints(bool hover)
```
and in line #289 the `getWaypoint()` iterate the state machine by calling `calculateWaypoint()`,and the state machine is updated by the override function `runCurrentState()` and `chooseNextState()`.
In `WaypointGenerator::runCurrentState()`, one of the 4 state is `runTryPath()`, which calls `getPathMsg()` to create the message that is sent to the UAV. And then transform the message type from `avoidance::waypointResult` to:
* `mavros_msgs::Trajectory`: When `COM_OBS_AVOID` is true, this message must published to PX4 or will get warning
```sh
WARN  [avoidance] Obstacle Avoidance system failed, loitering
```
* `geometry_msgs::PoseStamped`: This works in **offboard** and **auto** mode. So when doing **auto:takeoff** we should either copy the waypoint or just not publishing this in **auto** mode or it will conflict to trajectory message. This can be replaced with `mavros_msgs::PositionTarget` to enable yaw_rate control, and which only works in **offboard** mode.

Though there are 5 points in [mavros_msgs::Trajectory](http://docs.ros.org/en/api/mavros_msgs/html/msg/Trajectory.html) but only first one is used. See `avoidance::transformToTrajectory()`. And FCU will feedback its next desired position in topic `/mavros/trajectory/desired`, use this to be the goal input for the local planner to calculate new obstacle free path.
> Note: `safe_landing_planner` used a different state machine as `local_planner`

For mission mode to enable avoidance, the avoidance node subscribe `/mavros/mission/waypoints` to get the current state of the vehicle and use it for the planner to update the new plan, and the state is changing only if the MAV_CMD type is [MAV_CMD_DO_CHANGE_SPEED](https://mavlink.io/en/messages/common.html#MAV_CMD_DO_CHANGE_SPEED) in `avoidance_node.cpp` line # 168:
```cpp
void AvoidanceNode::missionCallback(const mavros_msgs::WaypointList& msg)
```
# motor_test
From the time writing, the timeout `-t` and power level `-p` option is only support by [Dshot](https://github.com/PX4/PX4-Autopilot/issues/16547#issuecomment-832560211), not by PWM, which only runs the motor for 1 sec with 100% power. To enable Dshot control, set `SYS_USE_IO` = 0 and set `DSHOT_CONFIG` to the highest speed supported by ESC, and then reboot. Connect ESC signal with the FCU port [FMU PWM OUT](https://docs.px4.io/master/en/peripherals/dshot.html#wiring-connections), other than `IO PWM OUT`.

# PID tuning
PID value of [F330](https://discuss.px4.io/t/f330-vibration-problem/16852). Avoid setting p gain too low, or the i gain too large to prevent the [integrator windup](https://discuss.px4.io/t/f330-vibration-problem/16852/4).
## lose height suddenly when hovering
https://discuss.px4.io/t/multicopter-drops-the-altitude-hold-in-posctl-video-attached/18600