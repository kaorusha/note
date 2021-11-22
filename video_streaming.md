# video streaming within local network (cctv)
when creating video we can use `v4l2src device=/dev/video0` to replace `raspivid`, and use `v4l2h264enc` to replace the deprecated `omxh264enc` as this complete and updated [example](https://qengineering.eu/install-gstreamer-1.18-on-raspberry-pi-4.html).
## gstreamer TCP
On raspberry pi zero as host, install `gstreamer` as the [tutorial](https://platypus-boats.readthedocs.io/en/latest/source/rpi/video/video-streaming-gstreamer.html). The host is server IP, and the server will before the client starts. 
```sh
raspivid -fps 26 -h 450 -w 600 -vf -n -t 0 -b 200000 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=x.x.x.x port=x
```
### bit rate
[Suggest](https://support.google.com/youtube/answer/1722171?hl=en#zippy=%2Cbitrate) 10Mbps for acceptable viewing quality for 1080p/30fps.
Delay time is less than 1 sec by 5M bit rate tested in the office, and the max bit rate is 25M. Setting 25M might lead to system crash.
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
## gstreamer UDP
The [legacy stack](https://www.raspberrypi.com/documentation/accessories/camera.html#libcamera-and-the-legacy-raspicam-camera-stack) performs better on raspberry pi 0 and 2 (GPU supported). Since Raspbian Bullseye it uses only [libcamera](https://www.raspberrypi.com/documentation/accessories/camera.html#network-streaming) as open source camera driver.
* RTP protocol [command](https://www.raspberrypi.com/documentation/accessories/camera.html#using-rtp) use the receiver's IP, and the server works regardless the receiver client is running or not.
> note: raspberry pi 0 will crush if using the default 25M bit rate.
```sh
# On server:
raspivid -n --inline -t 0 -b 5000000 -o - | gst-launch-1.0 fdsrc fd=0 ! h264parse ! rtph264pay ! udpsink host=x.x.x.x port=x
```
```sh
# On receiver client of another machine
gst-launch-1.0 udpsrc address=x.x.x.x port=x caps=application/x-rtp ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink
```
### view video with QGC, other than gstreamer app
Use the same port as set in QGC preference to see video with QGC flight view window.
> note: QGC use the last user changed gstream setting. QGC also support other [protocols](https://docs.qgroundcontrol.com/master/en/SettingsView/General.html#video).
### fps-update-interval
If setting `autovideosink fps-update-interval=1000`(meaning update the image every 1000ms) will get occasionally warning:
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
Capable of streaming through public IP, and store the data for playback.
RTMP and HLS were most used among the [Streaming protocols](https://jasonblog.github.io/note/media_player/streaming_tong_xun_xie_ding_rtp_rtcp_rtsp_rtmp_hls.html). See the history and property [compare of protocols](https://blog.csdn.net/leixiaohua1020/article/details/18893769). If the server is using public IP, then other device can access it as this [concept diagram](https://forums.raspberrypi.com/viewtopic.php?t=207639).
## push stream with encoder to server
* [ffmpeg](https://forums.raspberrypi.com/viewtopic.php?t=173230): widely used but relatively old encoder.
* [gstreamer](https://qengineering.eu/install-gstreamer-1.18-on-raspberry-pi-4.html): officially suggested by rpi camera document including a comprehensive tutorial and example for both 32 bit and 64 bit OS, as well as the **YouTube** streaming. `gstreamer` is [faster](https://blog.gtwang.org/iot/raspberry-pi-nginx-rtmp-server-live-streaming/) than `ffmpeg`, and support a lot of encoders, `v4l2h264enc` is the latest suggested, while other encoders, [avconv, omxh264enc, x264enc](https://dotblogs.com.tw/RichardNote/2018/10/29/002238), is compared by the streaming efficiency.  
> additional microphone is required or use 'dummy' audio stream for **YouTube** to accept the stream.
## server options
Video stream is pushed to a server for receivers to playback.
### AWS kinesis remote server
A serverless stream service means the server is running on cloud, so the maintaining and resource allocation of the server is taken care by the service supplier, which is the main advantage for companies with fast growing user data. AWS is one of the popular options, and the [cost](https://aws.amazon.com/tw/kinesis/video-streams/pricing/?nc=sn&loc=3) is based on the data flow. 
> note: **Ring** offer an optional monthly cost [subscription](https://ring.com/protect-plans) per device for customers video storage for maximum [60 days](https://support.ring.com/hc/en-us/articles/360047871752-Understanding-and-Adjusting-Your-Video-Storage-Time-).
### rtmp server with Nginx
Nginx is a popular open source web server software. [Enabling Video Streaming for Remote Learning with NGINX and NGINX Plus](https://www.nginx.com/blog/video-streaming-for-remote-learning-with-nginx/) is one of the use cases of its `rtmp-module`. It also support [Setting up HLS live streaming server using NGINX + nginx-rtmp-module on Ubuntu](https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/). [Create your own video streaming server with Linux](https://opensource.com/article/19/1/basic-live-video-streaming-server) with `BSD`, a server software support broadcasting stream, so the users can watch the stream with player software like `vlc`. There are also other option for broadcasting, see [8 Free & Best Open source Video Streaming Servers Software](https://www.how2shout.com/tools/free-best-open-source-video-streaming-servers-software.html).
The [docker](https://hub.docker.com/r/jasonrivers/nginx-rtmp) contains the above nginx and BSD settings.
```sh
# run server to receive the stream
sudo docker run -p1935:1935 -p8080:8080 jasonrivers/nginx-rtmp
```
push streaming from another machine with camera, use rtmp server's IP.
```sh
# longest delay time with this setting, too high bitrate occasionally shows up when streaming to youtube.
gst-launch-1.0 v4l2src device=/dev/video0 ! \
video/x-h264, width=1920, height=1080, framerate=30/1 ! h264parse ! \
mux. \
audiotestsrc wave=silence ! voaacenc bitrate=128000 ! aacparse ! \
mux. \
flvmux streamable=true name=mux ! queue ! \
rtmpsink location="rtmp://x.x.x.x/live/mystream" 
```
The following setting might be faster but it really depend on the web transmission speed.
```sh
# minimum delay time (faster without -e) using video_bitrate parameter to reduce bitrate, warning of too low bitrate occasionally shows up when streaming to youtube. 
gst-launch-1.0 -e v4l2src device=/dev/video0 ! \
video/x-raw,width=1920, height=1080, framerate=30/1 ! \
v4l2h264enc extra-controls="controls, h264_profile=4, video_bitrate=5000000" ! \
'video/x-h264, profile=high, level=(string)4' ! h264parse ! \
mux. \
audiotestsrc wave=silence ! voaacenc bitrate=128000 ! aacparse ! \
mux. \
flvmux streamable=true name=mux ! queue ! \
rtmpsink location="rtmp://x.x.x.x/live/mystream"
```
Then use `vlc` with rtmp server's ip to watch stream.
### rtsp server
docker [rtsp-simple-server](https://malagege.github.io/blog/2021/08/22/Raspberry-pi-%E6%94%9D%E5%BD%B1%E6%A9%9F%E8%A8%88%E7%95%AB/)