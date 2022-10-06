# rasp.local
Project page for my personal Raspberry Pi (Original Model B).

## Why?

My Raspberry Pi is very old and cannot run OctoPi.  As a compromise, I'm trying to build a lightweight solution for monitoring print jobs and (hopefully) starting/stopping print jobs remotely.

After all, my Pi is old, not broken :wink:

## Build the base image

Install the Raspberry Pi Imager and flash an SD card with Raspberry Pi OS Lite (32bit).  It is recommended to use key-based authentication for SSH.


## Installing TP-Link AC600 Nano driver on Raspbian (32bit)
The original Raspberry Pi Model B does not have a wifi card installed.  I chose the TP-Link AC600 because of its compact form factor and dual-band support. This section outlines how to install the device drivers for Archer T2U Nano.

**Install the build dependencies:**

`sudo apt install dkms git build-essential libelf-dev`



**Clone and build the driver:**

```bash
git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au/
sudo make dkms_install
```

\*_Note: The build may take several minutes to complete._

**Check to make sure the install was successful:**

`sudo dkms status`

You should see an entry for `8812au` with an `installed` status.