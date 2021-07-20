# Read sensor data from USB or Uart port of raspberry pi zero w
First check the communication protocols of sensor. The raspberry pi [pingout](https://pinout.xyz/pinout/pin10_gpio15#) shows GPIO supports different protocols such as UART/SPI/I2C from different pin. There are severial libraries and code languages for digital write/read data from GPIO, with the example and efficiency [comparison](https://codeandlife.com/2012/07/03/benchmarking-raspberry-pi-gpio-speed/).
## UART
The raspberry pi [UART config](https://www.raspberrypi.org/documentation/configuration/uart.md) listed that GPIO 14 and 15 is the primary UART and the default type is mini UART. The GPIO 14 and 15 by default is disabled and can be opened through `raspi-config`. The uart serial port default usage is outputing linux console (through usb connection, or RS232 monitor, can be replaced by SSH). To use uart port for sensor communacation, answer 'No' at the prompt `Would you like a login shell to be accessible over serial?` to disable Linux serial console. See the detail [config steps](https://dumbcatnote.blogspot.com/2020/04/raspberry-pi-enable-serial-port.html).
### read USB serial port
After enabling hardware ports, [udev library](https://manpages.debian.org/buster/libudev-dev/udev_device_get_parent_with_subsystem_devtype.3.en.html) is used for reading data from serial ports including USB and UART. For computers that only provides USB ports, this is helpful. We need to prepare a UART to USB transform adapter.
Udev is a dymamic list of all devices on the computer. [Use Udev for Device Detection and Management in Linux](https://www.tecmint.com/udev-for-device-detection-management-in-linux/) by adding ruls that can run scripts on adding and removeing device. The [example](https://blog.csdn.net/baidu_39220322/article/details/112537199) shows how to find the matched USB port to access the sensor data.
```sh
# result of PC
$ udevadm info -q all -n /dev/ttyUSB*
P: /devices/pci0000:00/0000:00:01.2/0000:02:00.0/usb1/1-4/1-4:1.0/ttyUSB0/tty/ttyUSB0
N: ttyUSB0
L: 0
S: serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0
S: serial/by-path/pci-0000:02:00.0-usb-0:4:1.0-port0
E: DEVPATH=/devices/pci0000:00/0000:00:01.2/0000:02:00.0/usb1/1-4/1-4:1.0/ttyUSB0/tty/ttyUSB0
E: DEVNAME=/dev/ttyUSB0
E: MAJOR=188
E: MINOR=0
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=14479996252
E: ID_BUS=usb
E: ID_VENDOR_ID=10c4
E: ID_MODEL_ID=ea60
E: ID_PCI_CLASS_FROM_DATABASE=Serial bus controller
E: ID_PCI_SUBCLASS_FROM_DATABASE=USB controller
E: ID_PCI_INTERFACE_FROM_DATABASE=XHCI
E: ID_VENDOR_FROM_DATABASE=Silicon Labs
E: ID_VENDOR=Silicon_Labs
E: ID_VENDOR_ENC=Silicon\x20Labs
E: ID_MODEL=CP2102_USB_to_UART_Bridge_Controller
E: ID_MODEL_ENC=CP2102\x20USB\x20to\x20UART\x20Bridge\x20Controller
E: ID_REVISION=0100
E: ID_SERIAL=Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001
E: ID_SERIAL_SHORT=0001
E: ID_TYPE=generic
E: ID_USB_INTERFACES=:ff0000:
E: ID_USB_INTERFACE_NUM=00
E: ID_USB_DRIVER=cp210x
E: ID_MODEL_FROM_DATABASE=CP210x UART Bridge
E: ID_PATH=pci-0000:02:00.0-usb-0:4:1.0
E: ID_PATH_TAG=pci-0000_02_00_0-usb-0_4_1_0
E: ID_MM_CANDIDATE=1
E: DEVLINKS=/dev/serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0 /dev/serial/by-path/pci-0000:02:00.0-usb-0:4:1.0-port0
E: TAGS=:systemd:
```
```sh
# result of raspberry pi zero w
$ udevadm info -q all -n /dev/ttyUSB*
P: /devices/platform/soc/20980000.usb/usb1/1-1/1-1:1.0/ttyUSB0/tty/ttyUSB0
N: ttyUSB0
L: 0
S: serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0
S: serial/by-path/platform-20980000.usb-usb-0:1:1.0-port0
E: DEVPATH=/devices/platform/soc/20980000.usb/usb1/1-1/1-1:1.0/ttyUSB0/tty/ttyUSB0
E: DEVNAME=/dev/ttyUSB0
E: MAJOR=188
E: MINOR=0
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=26078064
E: ID_VENDOR=Silicon_Labs
E: ID_VENDOR_ENC=Silicon\x20Labs
E: ID_VENDOR_ID=10c4
E: ID_MODEL=CP2102_USB_to_UART_Bridge_Controller
E: ID_MODEL_ENC=CP2102\x20USB\x20to\x20UART\x20Bridge\x20Controller
E: ID_MODEL_ID=ea60
E: ID_REVISION=0100
E: ID_SERIAL=Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001
E: ID_SERIAL_SHORT=0001
E: ID_TYPE=generic
E: ID_BUS=usb
E: ID_USB_INTERFACES=:ff0000:
E: ID_USB_INTERFACE_NUM=00
E: ID_USB_DRIVER=cp210x
E: ID_VENDOR_FROM_DATABASE=Cygnal Integrated Products, Inc.
E: ID_MODEL_FROM_DATABASE=CP2102/CP2109 UART Bridge Controller [CP210x family]
E: ID_PATH=platform-20980000.usb-usb-0:1:1.0
E: ID_PATH_TAG=platform-20980000_usb-usb-0_1_1_0
E: DEVLINKS=/dev/serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0 /dev/serial/by-path/platform-20980000.usb-usb-0:1:1.0-port0
E: TAGS=:systemd:
```
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

----
/dev/ttyAMA0    uart port
/dev/ttyS0    uart port
FOUND LiDAR_LD06
LiDAR_LD06 started successfully 
parse sec = 0.00278798	assem sec = 0.00209298
parse sec = 0.000441997	assem sec = 0.0606195
parse sec = 0.0163279	assem sec = 0.00247998
parse sec = 0.00872792	assem sec = 0.00835393
parse sec = 0.00787193	assem sec = 0.0829813
parse sec = 0.0327417	assem sec = 0.126448
parse sec = 0.0527035	assem sec = 0.174569
parse sec = 0.101655	assem sec = 0.311154
parse sec = 0.0906682	assem sec = 0.47239
parse sec = 0.0835903	assem sec = 0.65605
parse sec = 0.0827283	assem sec = 0.793427
parse sec = 0.0921162	assem sec = 0.974048
parse sec = 0.0821403	assem sec = 1.29107
parse sec = 0.0949982	assem sec = 1.58028
parse sec = 0.0847273	assem sec = 1.92614
parse sec = 0.104941	assem sec = 2.25621
parse sec = 0.0765083	assem sec = 2.62824
parse sec = 0.0769873	assem sec = 2.96672
parse sec = 0.0872573	assem sec = 3.30345
parse sec = 0.112692	assem sec = 3.55379
parse sec = 0.0728104	assem sec = 3.80667
parse sec = 0.102885	assem sec = 4.09439
parse sec = 0.0904103	assem sec = 4.37244
parse sec = 0.0790764	assem sec = 4.61368
parse sec = 0.0782694	assem sec = 4.8309
parse sec = 0.0689905	assem sec = 5.36024
parse sec = 0.0637765	assem sec = 7.60707
parse sec = 0.112511	assem sec = 7.47909
parse sec = 0.0790634	assem sec = 6.33042
parse sec = 0.102687	assem sec = 8.30927
parse sec = 0.100671	assem sec = 6.46453
parse sec = 0.0765754	assem sec = 6.7661
parse sec = 0.0765664	assem sec = 7.0719
parse sec = 0.113946	assem sec = 7.30474
parse sec = 0.0844714	assem sec = 10.2543
parse sec = 0.0994133	assem sec = 12.3118
parse sec = 0.186155	assem sec = 10.1638
parse sec = 0.1341	assem sec = 12.3651
parse sec = 0.132428	assem sec = 13.6215
parse sec = 0.0889924	assem sec = 9.4481
parse sec = 0.101103	assem sec = 13.6798
parse sec = 0.104178	assem sec = 14.336
parse sec = 0.118561	assem sec = 13.3767
parse sec = 0.101822	assem sec = 15.6149
parse sec = 0.135514	assem sec = 10.8727
parse sec = 0.0910915	assem sec = 10.333
parse sec = 0.0884025	assem sec = 10.7627
parse sec = 0.0990864	assem sec = 10.7752
parse sec = 0.0843085	assem sec = 11.1795
parse sec = 0.0909675	assem sec = 12.0503
^C[LD06-2] killing on exit
parse sec = 0.0737596	assem sec = 7.22158
[rosout-1] killing on exit
