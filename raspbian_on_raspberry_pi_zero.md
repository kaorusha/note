# raspbian on raspberry pi zero
## Installing raspbian OS
### headless SSH
* editing the `wpa_supplicant.conf` as the [instruction](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)
* SSH can be enabled by placing a file named `ssh` in /boot, as [this step](https://www.raspberrypi.org/documentation/remote-access/ssh/)
## Install ROS2 for raspberry pi zero w
[ros wiki](https://answers.ros.org/question/299588/can-ros2-run-on-raspberry-pi-zero-w/) suggest 2 method for runnung ROS2 on raspberry pi zero w: 
* **on Arch Linux**: could not build ros2-foxy through pacman, because fail of building fast-rtps
* **on Raspbian(Official OS for Raspberry Pi zero w)**

Both method highly suggest cross compiling (runs really really much faster)
> Another reference for installing ROS2 directory from source on Raspbian on [raspberry pi 4b](https://medium.com/swlh/raspberry-pi-ros-2-camera-eef8f8b94304)
### Cross compining on the top of raspbian OS
[cross compile env docker](https://github.com/cyberbotics/epuck_ros2/tree/master/installation/cross_compile): cross compiling ros2 packages
#### Cross compiling other packages
This [reference](https://raspberrypi.stackexchange.com/questions/103737/cross-compile-for-raspberry-pi-zero-from-ubuntu) answered cross compiler tools options for **raspberry pi zero**.
* [RaspberryPi toolchain](https://github.com/raspberrypi/tools): older gcc 4.9.3, [example](https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187) of compiling WiringPi library.
* [RaspberryPi toolchains v3](https://github.com/abhiTronix/raspberry-pi-cross-compilers): newer gcc(8.3 and later) and with complete detail [steps](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Cross-Compiler-CMake-Usage-Guide-with-rsynced-Raspberry-Pi-32-bit-OS#cross-compiler-cmake-usage-guide-with-rsynced-raspberry-pi-32-bit-os) for newbies. Note: Link Time Optimization [enable](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Cross-Compiler:-Installation-Instructions#d-advanced-information) and [Flags](https://github.com/abhiTronix/raspberry-pi-cross-compilers#optimization-flags-involved)
* [buildroot](https://buildroot.org/)
* [crosstool-NG](https://crosstool-ng.github.io/docs/introduction/): most popular and has a has a `armv6-rpi-linux-gnueabi` for raspberry pi zero (and w).