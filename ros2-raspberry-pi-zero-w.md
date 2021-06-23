personal note for installing Arch on **Raspberry Pi zero w**, and ROS2
# installing Arch Linux
Px4 user document recommended [ROS2](https://docs.px4.io/master/en/ros/ros2.html) for better support:
>This contrasts with ROS (1), which communicates with PX4 via MAVROS/MAVLink, hiding PX4's internal architecture and many of its conventions (e.g. frame and unit conversions).
>ROS 2 (and the bridge) will become easier to use as the development team provide ROS 2 APIs to abstract PX4 conventions, along with examples demonstrating their use. These are planned in the near-term PX4 roadmap.

The [raspberry-pi-zero-operating-system-list](https://www.kodifiretvstick.com/raspberry-pi-zero-operating-system-list/)  not including Ubuntu (official OS for ROS 2 [Foxy](https://docs.ros.org/en/foxy/Installation/Ubuntu-Development-Setup.html)), so using Arch Linux as alternative.
## host setup
Ubuntu 18.04, not VM (my VM can not read SD card) 
(this [repo](https://github.com/austinstig/ros2-raspberry-pi-zero-w) use VM to do crosscompiler) 
## create SD image
* follow official [installation](https://archlinuxarm.org/platforms/armv6/raspberry-pi) steps
before step 5, run
```sh
su -s
```
to [change to root](https://askubuntu.com/questions/617850/changing-from-user-to-superuser) while remain current shell config

before running `bsdtar`, run
```sh
bsdtar --version
```
and shows the following result on Ubuntu 18.04:
>bsdtar 3.2.2 - libarchive 3.2.2 zlib/1.2.11 liblzma/5.2.2 bz2lib/1.0.6 liblz4/1.7.1

follow this [repo](https://github.com/helotism/helotism/issues/8) to update the `bsdtar` to 3.3.x solve the following error:
>bsdtar: Ignoring malformed pax extended attribute </br>
>bsdtar: Ignoring malformed pax extended attribute </br>
>bsdtar: Error exit delayed from previous errors. </br>

before step 8, if using headless access:
## add wifi config without a monitor, so we can SSH headless
* follow the [steps](https://ladvien.com/installing-arch-linux-raspberry-pi-zero-w/)
* after connecting through SSH, complete step 9 and 10
* optional: add another wifi SSID in order to run headless at another position:
```sh
sudo -s
nano /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```
add multiple SSID as [raspberry doc](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)
* complete the rest steps.