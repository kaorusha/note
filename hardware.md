https://roverrobotics.com/

https://www.theconstructsim.com/82-available-ros2-hardware/

[indoor localization rf tags](https://idminer.com.tw/product/stargazer-%e5%ae%a4%e5%85%a7%e5%ae%9a%e4%bd%8d%e6%84%9f%e6%b8%ac%e5%99%a8/)

# camera solution
From the pp. 11 of [cm4 datasheet](https://datasheets.raspberrypi.org/cm4/cm4-datasheet.pdf):
> Camera sensors supported by the official Raspberry Pi firmware are; the OmniVision `OV5647`, Sony `IMX219` and Sony
`IMX477`, no security device is required on Compute Module devices to use these camera sensors.

The chips listed above corresponding to the native raspberry pi [cameras](https://www.raspberrypi.org/documentation/accessories/camera.html). Yet there are more options can be found on [ArduCam](https://www.arducam.com/raspberry-pi-camera-solution/), including [20MP camera](https://www.arducam.com/product/arducam-20mp-imx283-camera-module-with-m12-mount-lens-and-adapter-board-for-depthai/), [MIPI Camera Modules](https://www.arducam.com/docs/cameras-for-raspberry-pi/mipi-camera-modules/), as well as information of [Connector Type](https://www.arducam.com/raspberry-pi-camera/connector-type-pinout/) and [Multi-Camera Adapter Board](https://www.arducam.com/docs/cameras-for-raspberry-pi/multi-camera-adapter-board/multi-camera-adapter-board-v2-1/). 

And there is also another 20MP camera module, [MER-2000-19U3M/C](https://www.daheng-imaging.com/products/ProductDetails.aspx?current=5&productid=2941), that use `USB3.0` to connect with raspberry pi. The camera module is similer to drone camera (a seperate camera module). From the [original post](https://forum.allaboutcircuits.com/threads/20mp-camera-on-raspberry-pi.151574/) the guy used `MER-2000-19U3C` which is not available in our region but the official website provides connecting [guild](https://www.get-cameras.com/Raspberry-Pi-with-20MP-industrial-camera) for [USB3](https://www.get-cameras.com/FAQ-ARM-Board-WITH-USB3-Camera).

common action camera for drone:
* [GoPro hero 5, hero 4](https://dronesplayer.com/drones/gopro-karma-drone/)
* [RunCam 2](https://shop.runcam.com/runcam2/)
* [Foxeer legend 2](https://www.foxeer.com/foxeer-legend-2-uhd-camera-1080p-60fps-racing-drone-g-2)

## FPV camera, action camera
Main difference of [FPV](https://www.dronezon.com/learn-about-drones-quadcopters/what-is-fpv-camera-fov-tvl-cmos-ccd-technology-in-drones/)(cited from [here](https://www.dronetrest.com/t/fpv-cameras-for-your-drone-what-you-need-to-know-before-you-buy-one/1441)) camera and action camera:
| Item | FPV | action |
| :----:| :----: | :----: |
| Delay | around 40ms | most >100ms |
| Resolution | low | high |
| Dimension | 20mm(L) * 20mm(W) * 20mm(H) similar | 66mm(L) * 38mm(W) * 17mm(H) similar |
| Weight | < 10g | around 50g |
| Supplier | swift, eagle, predator, sparrow ([compare](https://www.youtube.com/watch?v=f-Wk3_4T6rg)) | [GoPro](https://youtu.be/yWv5Sp0u0VA), [RunCam 2 HD, Foxeer Legend 1](https://www.dronetrest.com/t/runcam-2-hd-vs-foxeer-legend-1/1479) |
# Video streaming
A brief [introduction](https://www.dronezon.com/learn-about-drones-quadcopters/learn-about-uav-antenna-fpv-live-video-transmitters-receivers/), [FPV Gear Guide](https://www.dronetrest.com/t/fpv-gear-guide-overview-on-what-to-buy/118) and recommended [receivers, monitors, and goggles](https://www.dronetrest.com/t/fpv-receivers-monitors-and-goggles-which-one-should-i-use/1505)
## Tracking antenna
The high end [Mobile UAV Tracking Antenna](https://uavfactory.com/en/products/tracking-antenna/portable-tracking-antenna) has a range of 62 miles (100 km).  Another is the [Veronte antenna tracker](https://www.embention.com/news/tracking-antennas-long-range-drones-uas/).
## Antenna
[types and wave shape](https://www.dronetrest.com/t/the-complete-guide-to-fpv-antennas-for-your-drone/1473). The helical antenna can make both omni-directional as rubber duck antenna or directional by changing the length and pitch.
## Diversity Controllers
Installed with receiver. Switch between omni-directional antenna and directional antenna signal. 
[FR632](https://www.amazon.com/FR632-Raceband-Wireless-Diversity-FPV/dp/B019PZNF48/ref=as_li_ss_tl?keywords=Dual+Wireless+Diversity+Receiver+for+FPV&qid=1581621881&sr=8-3&linkCode=sl1&tag=ph4store-20&linkId=dd574a61e21d0a5252c5fb6c62f16447)
## Power level
5.8GHz is allowed under **FCC**(<30dBm) and **SRRC**(<28dBm), while **MIS** and **CE** only allow 2.4Ghz. [Transmitter buying guide](https://www.dronetrest.com/t/fpv-video-transmitter-buying-guide/1470) suggest 5.8GHz to get further distance. See [compare](https://www.imnobby.com/2020/02/08/dji-mavic-mini-%E5%90%84%E5%9C%B0%E5%8D%80%E5%9E%8B%E8%99%9F%E7%89%88%E6%9C%AC%E6%AF%94%E8%BC%83/). While some [DIY](https://flzen.wordpress.com/2020/08/09/dji-mini-channel-band/) users tend to modify with 2.4Ghz to get better penetration performance.
### Video transmitter TX
Need Both 2.4Ghz and 5.8GHz transmitter on drone. There are hundreds of this transmitter board, some options:
* [FT951 5.8GHz](https://www.amazon.in/HobbyKing-FT951-5-8GHz-Transmitter-Certification/dp/B01MEGK6V3)
* [TS832](https://www.amazon.com/TS832-Transmitter-Wireless-Module-Racing/dp/B06XKQ8466)
* [250mW 5G8 7CH TX](https://www.fatshark.com/product/250mw-5g8-transmitter-v3/)
* [2.4 Ghz transmitter](https://www.defiancerc.com/products/furious-fpv-stealth-2-4ghz-long-range-video-transmitter?variant=12729497845829)
# flash lidar
[comparison](https://www.dronezon.com/learn-about-drones-quadcopters/best-uses-for-time-of-flight-tof-camera-depth-sensor-technology-in-drones-or-ground-based/)

# distance sensor (rangefinder)
## vl53l1x
The driver in PX4 `vl53l1.cpp` checks the status with `VL53L1X_GetRangeStatus()`. Status other than 0 will not publish distance value to PX4, so from **mavshell** the `listener` will get the last valid measurement. The raw value is checked by threshold value to filtered out [modulation problem](https://community.st.com/s/question/0D50X00009sUiJUSA0/out-of-range-readings-of-vl53l1x). 
# PCB antenna
# Frame
## FPV
* [Kopis CineWhoop 3"](https://shop.holybro.com/spare-parts-kopis-cinewhoop_p1199.html) with [Tmotor F1507 kv3800(Stardard)](https://www.getfpv.com/t-motor-f1507-v1-2700kv-3800kv-motor.html) with T3140 propeller: 50% throttle 238.14g * 4 - 237g frame - 150g [battery](https://www.ruten.com.tw/item/show?22033087757284) = 565.6g for the rest devices
* [Kopis cinewhoop 2.5"](https://shop.holybro.com/kopis-cinewhoop25quot-kiss-aio-hd-polar-free-shipping_p1293.html) with [Tmotor F1404 KV3800](https://www.aliexpress.com/item/1005001994612066.html) with GF3016 propeller (not official Gemfan D63-5B which might be higher thrust with 5 blades): 50% throttle 158.01g * 4 - 155.2g frame - 101.6g [battery](https://shopee.tw/格氏原廠-TATTU-3S-4S-650-850mAh-14.8V-75C-暴力電池-i.5198380.7441715740) = 375g for the rest devices. (Take-off Weight: 249g from website)
* [MindRacer](http://drupal.xitronet.com/?q=catalog/20)
* [Kopis2 6S V2 5"](https://shop.holybro.com/kopis2-6s-v2free-shipping_p1169.html)
# ESC
[BLHeli-S ESC 20A](https://shop.holybro.com/spare-parts-x500-kit_p1252.html) Supports Dshot150, Dshot300 and Dshot600.