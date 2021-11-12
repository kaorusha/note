# video streaming
## gstream TCP
On raspberry pi zero as host, install `gstream` as the [tutorial](https://platypus-boats.readthedocs.io/en/latest/source/rpi/video/video-streaming-gstreamer.html). The host is server IP, and the server will before the client starts. 
```sh
raspivid -fps 26 -h 450 -w 600 -vf -n -t 0 -b 200000 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=x.x.x.x port=x
```
Delay time is less than 1 sec by 5M bit rate tested in the office, and the max bit rate is 25M. The doc suggest 15M for acceptable viewing quality.
```sh
# default using HD 1080p/30fps
raspivid -n -t 0 -b 5000000 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=x.x.x.x port=x
```
On receiving PC as client:
>  WARNING: erroneous pipeline: no element "ffdec_h264"
Because `ffmpeg` is deprecated, use `avdec` command as this [post](https://www.linkedin.com/pulse/streaming-live-from-pi-camerawith-raspberry-zero-w-pc-mundra/)
```sh
gst-launch-1.0 -v tcpclientsrc host=x.x.x.x port=x ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false
```
## gstream UDP
The [legacy stack](https://www.raspberrypi.com/documentation/accessories/camera.html#libcamera-and-the-legacy-raspicam-camera-stack) performs better on raspberry pi 0 and 2 (GPU supported). Since Raspbian Bullseye it uses only [libcamera](https://www.raspberrypi.com/documentation/accessories/camera.html#network-streaming) as open source camera driver.
* RTP protocol [command](https://www.raspberrypi.com/documentation/accessories/camera.html#using-rtp) use the client's IP, and the server works regardless the client is running or not.
> note: raspberry pi 0 will crush if using the default 25M bit rate.
```sh
# On server:
raspivid -n --inline -t 0 -b 5000000 -o - | gst-launch-1.0 fdsrc fd=0 ! h264parse ! rtph264pay ! udpsink host=x.x.x.x port=x
```
```sh
# On client of another machine
gst-launch-1.0 udpsrc address=x.x.x.x port=x caps=application/x-rtp ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink
```