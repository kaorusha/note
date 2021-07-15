# Read sensor data from USB or Uart port of raspberry pi zero w
First check the communication protocols of sensor. The raspberry pi [pingout](https://pinout.xyz/pinout/pin10_gpio15#) shows GPIO supports different protocols such as UART/SPI/I2C from different pin. There are severial libraries and code languages for digital write/read data from GPIO, with the example and efficiency [comparison](https://codeandlife.com/2012/07/03/benchmarking-raspberry-pi-gpio-speed/).
## UART
The raspberry pi [UART config](https://www.raspberrypi.org/documentation/configuration/uart.md) listed that GPIO 14 and 15 is the primary UART and the default type is mini UART. The GPIO 14 and 15 by default is disabled and can be opened through `raspi-config`. The uart serial port default usage is outputing linux console (through usb connection, or RS232 monitor, can be replaced by SSH). To use uart port for sensor communacation, answer 'No' at the prompt `Would you like a login shell to be accessible over serial?` to disable Linux serial console. See the detail [config steps](https://dumbcatnote.blogspot.com/2020/04/raspberry-pi-enable-serial-port.html).
### read USB serial port
After enabling hardware ports, [udev library](https://manpages.debian.org/buster/libudev-dev/udev_device_get_parent_with_subsystem_devtype.3.en.html) is used for reading data from serial ports including USB and UART. For computers that only provides USB ports, this is helpful. We need to prepare a UART to USB transform adapter.
Udev is a dymamic list of all devices on the computer. [Use Udev for Device Detection and Management in Linux](https://www.tecmint.com/udev-for-device-detection-management-in-linux/) by adding ruls that can run scripts on adding and removeing device. The [example](https://blog.csdn.net/baidu_39220322/article/details/112537199) shows how to find the matched USB port to access the sensor data.
### read UART serial port
If it is a USB than the `SUBSYSTEM`=usb, `DEVTYPE`=usb_device, but some device, such as ethernet and uart port, does not config `DEVTYPE`:
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
### Change baudrate
Shows the baud rate of a setial port:
```sh
stty -F /dev/ttyS0
```
The available baudrate setting [options](https://raspberrypi.stackexchange.com/questions/9422/can-a-raspberry-pi-be-operated-at-more-than-115200-baud-rate-i-e-230400).
Different methods can be used to change baudrates:
* C library `termios.h` has a solution for an arbitrary (non-standard) baud rate. See [config example](https://stackoverflow.com/questions/12646324/how-can-i-set-a-custom-baud-rate-on-linux/21960358#21960358).
Before running the set speed method, the result of /dev/ttyS0 is 9600.
After running the set speed method, the result:
```sh
speed 230400 baud; line = 0;
min = 0; time = 0;
-brkint -icrnl -imaxbel
-opost
-isig -icanon -iexten -echo -echoe -echok
```
* Python library `serial` [example](http://www.python-exemplary.com/drucken.php?inhalt_mitte=raspi/en/serial.inc.php)
* Modify the `/boot/config.txt` as [this](https://stackoverflow.com/questions/51234573/changing-the-baud-rate-of-a-serial-port-on-a-raspberry-pi-3) does the same. Doing POSIX manipulations first, then this to set the custom speed, works fine on the built-in UART of the Raspberry Pi to get a 250k baud rate. So as well as the POSIX standard rates, you can have pretty much any integer factor of 16M, up to at least 1M like [this](https://fw.hardijzer.nl/?p=138).