[udev library](https://manpages.debian.org/buster/libudev-dev/udev_device_get_parent_with_subsystem_devtype.3.en.html) is used for reading data from sensor driver. 
Udev is a dymamic list of all devices on the computer. [Use Udev for Device Detection and Management in Linux](https://www.tecmint.com/udev-for-device-detection-management-in-linux/) by adding ruls that can run scripts on adding and removeing device.

If it is a USB than the `SUBSYSTEM`=usb, `DEVTYPE`=usb_device, but some device, such as ethernet and serial port, does not config `DEVTYPE`:
```sh
# show information of devices
$ udevadm info -q all -n /dev/serial*

P: /devices/platform/soc/20215040.serial/tty/ttyS0
N: ttyS0
L: 0
S: serial0
E: DEVPATH=/devices/platform/soc/20215040.serial/tty/ttyS0
E: DEVNAME=/dev/ttyS0
E: MAJOR=4
E: MINOR=64
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=14693143
E: DEVLINKS=/dev/serial0
E: TAGS=:systemd:

P: /devices/platform/soc/20201000.serial/tty/ttyAMA0
N: ttyAMA0
L: 0
S: serial1
E: DEVPATH=/devices/platform/soc/20201000.serial/tty/ttyAMA0
E: DEVNAME=/dev/ttyAMA0
E: MAJOR=204
E: MINOR=64
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=14513265
E: DEVLINKS=/dev/serial1
E: TAGS=:systemd:
```
can not get another serial port
>dev path = /dev/ttyAMA0
[UART config](https://www.raspberrypi.org/documentation/configuration/uart.md)
[pingout](https://pinout.xyz/pinout/pin10_gpio15#)
[example](https://dumbcatnote.blogspot.com/2020/04/raspberry-pi-enable-serial-port.html)
change baudrate: