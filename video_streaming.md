# video streaming within local network (cctv)
## gstream TCP
On raspberry pi zero as host, install `gstream` as the [tutorial](https://platypus-boats.readthedocs.io/en/latest/source/rpi/video/video-streaming-gstreamer.html). The host is server IP, and the server will before the client starts. 
```sh
raspivid -fps 26 -h 450 -w 600 -vf -n -t 0 -b 200000 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=x.x.x.x port=x
```
### bit rate
[Suggest](https://support.google.com/youtube/answer/1722171?hl=en#zippy=%2Cbitrate) 10Mbps for acceptable viewing quality for 1080p/30fps.
Delay time is less than 1 sec by 5M bit rate tested in the office, and the max bit rate is 25M.
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
### view video with QGC, other than gstream app
Use the same port as set in QGC preference to see video with QGC flight view window.
> note: QGC use the last gstream setting ran by terminal command.
### fps-update-interval
If setting `autovideosink fps-update-interval=1000` will get occasionally warning:
```sh
WARNING: from element /GstPipeline:pipeline0/GstUDPSink:udpsink0: Error sending UDP packets
Additional debug info:
gstmultiudpsink.c(729): gst_multiudpsink_send_messages (): /GstPipeline:pipeline0/GstUDPSink:udpsink0:
client x.x.x.x:5600, reason: Error sending message: Network is unreachable
```
## motion
A video stream and motion detection package for raspberry pi to become a surveillance camera with [config](https://raspberry-valley.azurewebsites.net/Streaming-Video-with-Motion/). Enable **port forwarding** of the wifi router to stream the video to public network.
> note: **Ring** doorbell works the same as above.  

## port forwarding to public network
This method requires the access as well as password of the wifi router in order to change the port config.

# video streaming server
Capable of streaming through public ip, and store the data for playback.
[Streaming protocols](https://jasonblog.github.io/note/media_player/streaming_tong_xun_xie_ding_rtp_rtcp_rtsp_rtmp_hls.html)
## AWS kinesis
[Cost](https://aws.amazon.com/tw/kinesis/video-streams/pricing/?nc=sn&loc=3) base on the data flow.
> note: **Ring** offer an optional monthly cost [subscription](https://ring.com/protect-plans) per device for customers video storage for maximum [60 days](https://support.ring.com/hc/en-us/articles/360047871752-Understanding-and-Adjusting-Your-Video-Storage-Time-).
## Nginx
[Enabling Video Streaming for Remote Learning with NGINX and NGINX Plus](https://www.nginx.com/blog/video-streaming-for-remote-learning-with-nginx/)
[Setting up HLS live streaming server using NGINX + nginx-rtmp-module on Ubuntu](https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/)
[Create your own video streaming server with Linux](https://opensource.com/article/19/1/basic-live-video-streaming-server)
[8 Free & Best Open source Video Streaming Servers Software](https://www.how2shout.com/tools/free-best-open-source-video-streaming-servers-software.html)