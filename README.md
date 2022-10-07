# rasp.local
Project page for my personal Raspberry Pi (Original Model B).

## Why?

My Raspberry Pi is very old and cannot run OctoPi.  As a compromise, I'm trying to build a lightweight solution for monitoring print jobs and (hopefully) starting/stopping print jobs remotely.

After all, my Pi is old, not broken :wink:

# Build the base image

Install the Raspberry Pi Imager and flash an SD card with Raspberry Pi OS Lite (32bit).  It is recommended to use key-based authentication for SSH.


# Installing TP-Link AC600 Nano driver on Raspbian (32bit)
The original Raspberry Pi Model B does not have a wifi card installed.  I chose the TP-Link AC600 because of its compact form factor and dual-band support. This section outlines how to install the device drivers for Archer T2U Nano.

**Install the build dependencies:**

```bash
sudo apt install dkms git build-essential libelf-dev
```



**Clone and build the driver:**

```bash
git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au/
sudo make dkms_install
```

\*_Note: The build may take a while to run (~1 hour for me).  Go grab some :coffee:_

**Check to make sure the install was successful:**

```bash
sudo dkms status
```

You should see an entry for `8812au` with an `installed` status.  If so, unplug the wifi module and plug it back in.  The green lights on the module should light up.

## Wifi Authentication

Use `raspi-config` to configure the wifi connection.  The pi should appear as a wifi client on the network about a minute after the config is saved.

# Configuring the Web Cam

I used the [Logitech HD Laptop Webcam C615](https://www.amazon.com/dp/B004YW7WCY) for the camera.  Plug it in, and verify that the device can be found:

```bash
lsusb
```

Make note of the camera properties (we'll use these later)

```bash
v4l2-ctl -V
```

## Setup the web steam

```bash
sudo apt-get install nginx libnginx-mod-rtmp
```

Modify the `/etc/nginx/nginx.conf` file and add the following block:

```
rtmp {
  server {
    listen 8082;
    chunk_size 4096;
    allow publish 127.0.0.1;
    deny publish all;
    application live {
        live on;
        record off;
    }
  }
}
```

Add a directory to hold our web page `/var/www/rasp.local/`, and add a simple index.html file.

Add an entry under `/etc/nginx/sites-available/rasp.local` for our website, with the following content:

```nginx
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  root /var/www/rasp.local;
  index index.html;
  location / {
    try_files $uri $uri/ =404;
  }
}
```

Add a symlink to this file within the `sites-enabled` directory,
and unlink the default entry:

```bash
ln -s /etc/nginx/sites-available/rasp.local /etc/nginx/sites-enabled/rasp.local
unlink /etc/nginx/sites-enabled/default
```


Launch the nginx service:

```bash
sudo systemctl start nginx.service
```

Confirm that nginx is listening on port 8082 and 80:

```bash
sudo netstat -plunt | grep -i listen
```

Steam the webcam to nginx:

```bash
ffmpeg -re -i /dev/video0 -c:v copy -f flv rtmp://127.0.0.1:8082/live/cam0.flv 
```

```bash
ffmpeg -y -f v4l2 -video_size 640x480 -framerate 25 -i /dev/video0 -vcodec h264  -f flv rtmp://127.0.0.1:8082/live/cam0.flv 
```

```bash
ffmpeg -f v4l2 -i /dev/video0 -preset ultrafast -tune zerolatency -vcodec libx264 -r 10 -b:v 512k -s 640x360 -f mpegts -flush_packets 0 udp://0.0.0.0:5000?pkt_size=1316
```

