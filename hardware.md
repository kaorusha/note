https://roverrobotics.com/

https://www.theconstructsim.com/82-available-ros2-hardware/

[indoor localization rf tags](https://idminer.com.tw/product/stargazer-%e5%ae%a4%e5%85%a7%e5%ae%9a%e4%bd%8d%e6%84%9f%e6%b8%ac%e5%99%a8/)

## camera solution
From the pp. 11 of [cm4 datasheet](https://datasheets.raspberrypi.org/cm4/cm4-datasheet.pdf):
> Camera sensors supported by the official Raspberry Pi firmware are; the OmniVision `OV5647`, Sony `IMX219` and Sony
`IMX477`, no security device is required on Compute Module devices to use these camera sensors.

The chips listed above corresponding to the native raspberry pi [cameras](https://www.raspberrypi.org/documentation/accessories/camera.html). Yet there are more options can be found on [ArduCam](https://www.arducam.com/raspberry-pi-camera-solution/), including [20MP camera](https://www.arducam.com/product/arducam-20mp-imx283-camera-module-with-m12-mount-lens-and-adapter-board-for-depthai/), [MIPI Camera Modules](https://www.arducam.com/docs/cameras-for-raspberry-pi/mipi-camera-modules/), as well as information of [Connector Type](https://www.arducam.com/raspberry-pi-camera/connector-type-pinout/) and [Multi-Camera Adapter Board](https://www.arducam.com/docs/cameras-for-raspberry-pi/multi-camera-adapter-board/multi-camera-adapter-board-v2-1/). 

And there is also another 20MP camera module, [MER-2000-19U3M/C](https://www.daheng-imaging.com/products/ProductDetails.aspx?current=5&productid=2941), that use `USB3.0` to connect with raspberry pi. The camera module is similer to drone camera (a seperate camera module). From the [original post](https://forum.allaboutcircuits.com/threads/20mp-camera-on-raspberry-pi.151574/) the guy used `MER-2000-19U3C` which is not available in our region but the official website provides connecting [guild](https://www.get-cameras.com/Raspberry-Pi-with-20MP-industrial-camera) for [USB3](https://www.get-cameras.com/FAQ-ARM-Board-WITH-USB3-Camera).