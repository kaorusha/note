# Installing raspbian OS

## Install ROS2 for raspberry pi zero w
[ros wiki](https://answers.ros.org/question/299588/can-ros2-run-on-raspberry-pi-zero-w/) suggest 2 method for runnung ROS2 on raspberry pi zero w: 
* **on Arch Linux**: could not build ros2-foxy through pacman, because fail of building fast-rtps
* **on Raspbian(Official OS for Raspberry Pi zero w)**

Both method highly suggest cross compiling (runs really really much faster)
> note: for installing ROS2 on Raspbian on [raspberry pi 4b](https://medium.com/swlh/raspberry-pi-ros-2-camera-eef8f8b94304)
## cross compining on the top of raspbian OS
[reference](https://github.com/cyberbotics/epuck_ros2/tree/master/installation/cross_compile)
## headless SSH
* editing the `wpa_supplicant.conf` as the [instruction](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)
* SSH can be enabled by placing a file named `ssh` in /boot, as [this step](https://www.raspberrypi.org/documentation/remote-access/ssh/)