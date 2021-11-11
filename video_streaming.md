# video streaming
## gstream TCP
On raspberry pi zero as host, install `gstream` as the [tutorial](https://platypus-boats.readthedocs.io/en/latest/source/rpi/video/video-streaming-gstreamer.html)
```sh
raspivid -fps 26 -h 450 -w 600 -vf -n -t 0 -b 200000 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=x.x.x.x port=x
```
On receiving PC as client:
>  WARNING: erroneous pipeline: no element "ffdec_h264"
Because `ffmpeg` is deprecated, use `avdec` command as this [post](https://www.linkedin.com/pulse/streaming-live-from-pi-camerawith-raspberry-zero-w-pc-mundra/)
```sh
gst-launch-1.0 -v tcpclientsrc host=x.x.x.x port=x ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false
```
## gstream UDP
