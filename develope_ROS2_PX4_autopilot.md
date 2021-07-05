# Setup and use ROS 2 with PX4
Compenion computer setup.
This document is the personal note as following [instructions](https://docs.px4.io/master/en/ros/ros2_comm.html#installation-setup) for doing setup. (The supporting OS is Ubuntu 18.04 or higher, so some additional package needed for **raspberry pi zero w** with **Raspbian** OS).

Px4 user document recommended [ROS2](https://docs.px4.io/master/en/ros/ros2.html) for better support:
>This contrasts with ROS (1), which communicates with PX4 via MAVROS/MAVLink, hiding PX4's internal architecture and many of its conventions (e.g. frame and unit conversions).
>ROS 2 (and the bridge) will become easier to use as the development team provide ROS 2 APIs to abstract PX4 conventions, along with examples demonstrating their use. These are planned in the near-term PX4 roadmap.

## Fast DDS Installation
see [PX4 doc](https://docs.px4.io/master/en/dev_setup/fast-dds-installation.html)
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
#### Foonathan memory
For raspbian OS, install git and cmake first:
```sh
sudo apt update
sudo apt-get install git
sudo apt install -y cmake
cmake --version
``` 
### Fast DDS Installation from Sources
* (on raspbian)Install libssl for raspbian to solve the following error:
```sh
sudo apt install libssl-dev
```
>CMake Error at /usr/share/cmake-3.16/Modules/FindPackageHandleStandardArgs.cmake:146 (message):
>  Could NOT find OpenSSL, try to set the path to OpenSSL root folder in the
>  system variable OPENSSL_ROOT_DIR (missing: OPENSSL_CRYPTO_LIBRARY
>  OPENSSL_INCLUDE_DIR)
* (on raspbian)Install gtest and gmock for raspbian:
```sh
sudo apt-get install libgtest-dev
sudo apt-get install libgmock-dev
```
* Do the rest cmake as the [instruction](https://docs.px4.io/master/en/dev_setup/fast-dds-installation.html#fast-rtps-dds)