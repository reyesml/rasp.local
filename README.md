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

Disconnect the Pi's ethernet cable, then veryify that you can still connect to it via ssh.

# Configuring the Webcam

I used the [Logitech HD Laptop Webcam C615](https://www.amazon.com/dp/B004YW7WCY) for the camera.  Plug it in, and verify that the device can be found:

```bash
lsusb
```

## Set up the Stream

 I used mjpg-streamer as my stream server.  mjpg-streamer is a lightweight streaming application specifically designed for machines with limited resources.  It is perfect for my 700Mhz raspberry pi, and typically consumes less than 15% of the available CPU when streaming at 15fps.

 Follow the setup instructions in [mjpg-steamer](https://github.com/jacksonliam/mjpg-streamer).

After the package has been built, cd into `mjpg-streamer/mjpg-streamer-experimental` and run:

```bash
export LD_LIBRARY_PATH=.
./mjpg_streamer -o 'output_http.so -l 0.0.0.0 -p 8083' -i 'input_uvc.so -f 15 -r 640x480'
```

This launches a webserver listening to port 8083.  Next, open a web browser and navigate to `http://<hostname_or_IP>:8083/?action=stream`.  You should see the camera streaming to the browser :thumbsup:

## Tuning the webcam

mjpg_streamer supports configuring the framerate and resolution via the `-f` and `-r` options, but sometimes this isn't enough.  To discover more camera options, run `v4l2-ctl -l` to list the available options:

```plaintext
                     brightness 0x00980900 (int)    : min=0 max=255 step=1 default=128 value=128
                       contrast 0x00980901 (int)    : min=0 max=255 step=1 default=32 value=32
                     saturation 0x00980902 (int)    : min=0 max=255 step=1 default=32 value=32
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
                           gain 0x00980913 (int)    : min=0 max=255 step=1 default=64 value=65
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=2 value=2
      white_balance_temperature 0x0098091a (int)    : min=2800 max=6500 step=1 default=4000 value=3320 flags=inactive
                      sharpness 0x0098091b (int)    : min=0 max=255 step=1 default=22 value=22
         backlight_compensation 0x0098091c (int)    : min=0 max=1 step=1 default=1 value=1
                  exposure_auto 0x009a0901 (menu)   : min=0 max=3 default=3 value=3
              exposure_absolute 0x009a0902 (int)    : min=3 max=2047 step=1 default=166 value=415 flags=inactive
         exposure_auto_priority 0x009a0903 (bool)   : default=0 value=1
                   pan_absolute 0x009a0908 (int)    : min=-36000 max=36000 step=3600 default=0 value=0
                  tilt_absolute 0x009a0909 (int)    : min=-36000 max=36000 step=3600 default=0 value=0
                 focus_absolute 0x009a090a (int)    : min=0 max=255 step=17 default=51 value=85 flags=inactive
                     focus_auto 0x009a090c (bool)   : default=1 value=1
                  zoom_absolute 0x009a090d (int)    : min=1 max=5 step=1 default=1 value=1
```

For example, to disable the auto focus and adjust it manually, run the following commands:

```bash
v4l2-ctl -c focus_auto=0
v4l2-ctl -c focus_absolute=$((17 * 6))
```
