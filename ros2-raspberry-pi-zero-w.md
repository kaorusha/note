personal note for installing Arch Linux on **Raspberry Pi zero w**, and ROS2
# installing Arch Linux
Px4 user document recommended [ROS2](https://docs.px4.io/master/en/ros/ros2.html) for better support:
>This contrasts with ROS (1), which communicates with PX4 via MAVROS/MAVLink, hiding PX4's internal architecture and many of its conventions (e.g. frame and unit conversions).
>ROS 2 (and the bridge) will become easier to use as the development team provide ROS 2 APIs to abstract PX4 conventions, along with examples demonstrating their use. These are planned in the near-term PX4 roadmap.

The [raspberry-pi-zero-operating-system-list](https://www.kodifiretvstick.com/raspberry-pi-zero-operating-system-list/)  not including Ubuntu (official OS for ROS 2 [Foxy](https://docs.ros.org/en/foxy/Installation/Ubuntu-Development-Setup.html)), so using Arch Linux as alternative.
> note: for installing ROS2 on Raspbian (Official OS for Raspberry Pi zero w) [instruction](https://medium.com/swlh/raspberry-pi-ros-2-camera-eef8f8b94304)
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
> ./al-wpa-setup.sh: 5: ./al-wpa-setup.sh: [[: not found </br>
> ./al-wpa-setup.sh: 14: ./al-wpa-setup.sh: [[: not found </br>
> ./al-wpa-setup.sh: 19: ./al-wpa-setup.sh: [[: not found </br>
use bash not instead of default sh to solve above error
```
bash al-wpa-setup.sh /dev/sdx SSID PASSWORD
```
* after connecting through SSH, complete step 9 and 10
* optional: add another wifi SSID in order to run headless at another position:
```sh
sudo -s
nano /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```
add multiple SSID as [raspberry doc](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)
* complete the rest steps.
> Note: Raspberry Pi zero w does not supporting 5G hz wifi
## Installing base packages
* install pacman [base-devel](https://wiki.archlinux.org/title/Arch_User_Repository#Getting_started)
```sh
pacman -S --needed base-devel
# updating
pacman -Syu
```
* install git and sudo
```sh
pacman -S git sudo
```
* add user in the wheel group(see all [group list](https://wiki.archlinux.org/title/users_and_groups)) in order to use sudo command
[reference1](https://linoxide.com/add-user-to-sudoers-or-sudo-group-arch-linux/)
[reference2](https://www.gushiciku.cn/pl/g4ZU/zh-tw)
```
usermod  -G wheel alarm(default)
# change to root
su
chmod +w /etc/sudoers
nano sudoers
```
uncomment the line `#%wheel ALL=(ALL) ALL` as `%wheel ALL=(ALL) ALL` . Save and exit
```sh
chmod -w /etc/sudoers
exit 
# now the sudo command is enabled for the user
```
## Install AUR helper
The [instruction](https://wiki.archlinux.org/index.php/ROS#ROS_2) lists the dependencies on AUR package site.
Since these dependencies included AUR packages which can't be installed through `pacman`, installing one of [AUR helpers](https://wiki.archlinux.org/title/AUR_helpers) for saving time from manual cloning and installing one by one. 
> Warn: Users are responsible for checking safety of these AUR packages.
* install [yay](https://aur.archlinux.org/packages/yay/)
```sh
# a directory to managing downloaded AUR packages
mkdir AUR
cd AUR/
git clone https://aur.archlinux.org/yay.git
# this will also install go
cd yay/
makepkg -si
cd ../
```
## Config Locale and Git 
* change locale setting to support UTF-8 for solving following error
>   -> ERROR: Locale must support UTF-8. See https://wiki.archlinux.org/index.php/locale or https://wiki.archlinux.org/index.php/locale </br>
> error making: ros2-foxy
* config git for solving following error
```sh
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
```
> fatal: unable to auto-detect email address (got 'alarm@alarmpi.(none)') </br>
> ==> ERROR: A failure occurred in prepare(). </br>
>    Aborting... 
## Install ROS2 (not successful)
### install [ros2-arch-deps](https://aur.archlinux.org/packages/ros2-arch-deps/)
```sh
yay -S ros2-arch-deps
```
### Fast-rtps 
the fast rtps related packages can not build together with `ros2-foxy`, need pre-installation as [manual instruction](https://fast-rtps.docs.eprosima.com/en/v2.0.0/installation/sources/sources.html#manual-installation) steps.
> note: need `sudo` install
* install [ros2-foxy](https://aur.archlinux.org/packages/ros2-foxy/)
```sh
yay -S ros2-foxy
```
Before `makepkg` started, the `sudo ` asked the password. Than starts building.
>Get error:
>Failed   <<< fastrtps [5h 0min 19s, exited with code 2]
>                                             
>Summary: 42 packages finished [5h 55min 54s]
>  1 package failed: fastrtps
>  3 packages had stderr output: fastcdr fastrtps foonathan_memory_vendor
>  266 packages not processed
>==> ERROR: A failure occurred in build().
>    Aborting...

## cross compining on the top of raspbian OS
[reference](https://answers.ros.org/question/299588/can-ros2-run-on-raspberry-pi-zero-w/)
### headless SSH
* editing the `wpa_supplicant.conf` as the [instruction](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)
* SSH can be enabled by placing a file named `ssh` in /boot, as [this step](https://www.raspberrypi.org/documentation/remote-access/ssh/)