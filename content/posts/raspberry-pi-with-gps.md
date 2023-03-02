---
title: "Raspberry Pi With Gps"
date: 2017-05-22T16:40:31-06:00
draft: false
type: "post"
tags:
  - nextcloud
  - linux
  - php
  - gps
  - raspberry pi
  - python
  - biking
aliases:
  - /blog/raspberry-pi-with-gps
---

After receiving a Raspberry Pi Zero for Christmas, I struggled to find a project to use it with. I started looking around and came across the [MagPi](https://www.raspberrypi.org/magpi/) issues that listed project ideas for the Zero and Zero W (a Zero with built-in Wi-FI and Bluetooth). In [Issue 40](https://www.raspberrypi.org/magpi/issues/40/), there was a tutorial on creating a GPS Logger with a Zero. I was already on a path to take more control of my own data and tracking bike routes was one thing that was escaping me. Now I had a solution! I hit a few bumps along the way so I wanted to document them for others.

![Closed Pelican case with Raspberry Pi W and GPS componenets](/images/raspberry-pi-gps-color-closed.jpg)
![Opened Pelican case with Raspberry Pi W and GPS componenets](/images/raspberry-pi-gps-color-closed.jpg)

## Parts

* [GlobalSat BU-353-S4 USB GPS Receiver](http://amzn.to/2rDX1zH) (since it seemed to work based on the article)
* [Raspberry Pi Zero W](http://amzn.to/2qIJqHO) (a Raspbery Pi Zero would also work, but allows easier access to the GPS data since you can enable SSH access without additional dongles)
* [Pelican 1040 Micro Case](http://amzn.to/2qCd8zM)
* [Micro USB adapters](http://amzn.to/2qCdKFW)
* USB battery pack (I ended up using [one from SCOSCHE](http://amzn.to/2rDwuCw) that I had lying around)
* [microSD card](http://amzn.to/2rCIZz3)
* A few extra things for fun
  * [LoveRPi MicroUSB Push On Off Switch](http://amzn.to/2qJrCy3)
  * [90 Degree USB cables](http://amzn.to/2rKrfBW)

## Setup

I loaded a fresh copy of Raspbian to the microSD card and booted up. I enabled SSH access and changed the default `pi` user's password. Go to the Applications menu (the Raspberry Pi logo) > Preferences > Raspberry Pi Configuration. On the **System** tab, choose **Change Password**. On the **Interfaces** tab, enable **SSH**. Click **OK**.

Before starting to work with the GPS receiver, I opened the terminal and updated the OS.

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

The first step in the article was to run `tail` on `syslog` to see if Raspbian detects the GPS receiver. In the terminal run:

```
$ tail -f /var/syslog
```

The -f option tells tail to continue printing new lines to the screen.

I plugged in the GPS receiver. I should have seen something that told me the address of the USB device, but instead I got:

```
May  6 14:50:41 raspberrypi kernel: [ 2933.303857] usb 1-1.1: new full-speed USB device number 6 using dwc_otg
May  6 14:50:41 raspberrypi kernel: [ 2933.426237] usb 1-1.1: New USB device found, idVendor=067b, idProduct=2303
May  6 14:50:41 raspberrypi kernel: [ 2933.426272] usb 1-1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
May  6 14:50:41 raspberrypi kernel: [ 2933.426287] usb 1-1.1: Product: USB-Serial Controller D
May  6 14:50:41 raspberrypi kernel: [ 2933.426300] usb 1-1.1: Manufacturer: Prolific Technology Inc. 
May  6 14:50:41 raspberrypi systemd-udevd[10099]: failed to execute '/lib/udev/mtp-probe' 'mtp-probe /sys/devices/platform/soc/20980000.usb/usb1/1-1/1-1.1 1 6': No such file or directory
```

Note the last line of output. Raspbian was missing `mtp-probe` in order to identify the GPS receiver as an [MTP](https://en.wikipedia.org/wiki/Media_Transfer_Protocol) device.

Exit out of `tail` with `Ctrl + C` and unplug the GPS receiver. To fix the error, simply install `libmtp-runtime`.

```
$ sudo apt-get install libmtp-runtime
```

Before continuing, verify `/lib/udev/mtp-probe` exists and is executable:

```
$ ls -l /lib/udev/
...
-rwxr-xr-x 1 root root    9684 Nov 15  2014 mtp-probe
```

I tried plugging in the GPS receiver at that point, but still didn't get the output I expexted:

```
May 11 21:39:21 raspberrypi kernel: [ 2448.869590] usb 1-1.1: new full-speed USB device number 5 using dwc_otg
May 11 21:39:21 raspberrypi kernel: [ 2448.992614] usb 1-1.1: New USB device found, idVendor=067b, idProduct=2303
May 11 21:39:21 raspberrypi kernel: [ 2448.992656] usb 1-1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
May 11 21:39:21 raspberrypi kernel: [ 2448.992678] usb 1-1.1: Product: USB-Serial Controller D
May 11 21:39:21 raspberrypi kernel: [ 2448.992696] usb 1-1.1: Manufacturer: Prolific Technology Inc. 
May 11 21:39:21 raspberrypi mtp-probe: checking bus 1, device 5: "/sys/devices/platform/soc/20980000.usb/usb1/1-1/1-1.1"
May 11 21:39:21 raspberrypi mtp-probe: bus: 1, device: 5 was not an MTP device
```

Restart Raspbian. After all, restarting fixes everything, right? :)

After restarting, run `tail -f /var/syslog` again and plugin the GPS receiver.

```
May 11 21:48:32 raspberrypi kernel: [   85.341026] usb 1-1.1: new full-speed USB device number 4 using dwc_otg
May 11 21:48:33 raspberrypi kernel: [   85.473822] usb 1-1.1: New USB device found, idVendor=067b, idProduct=2303
May 11 21:48:33 raspberrypi kernel: [   85.473847] usb 1-1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
May 11 21:48:33 raspberrypi kernel: [   85.473862] usb 1-1.1: Product: USB-Serial Controller D
May 11 21:48:33 raspberrypi kernel: [   85.473874] usb 1-1.1: Manufacturer: Prolific Technology Inc. 
May 11 21:48:33 raspberrypi mtp-probe: checking bus 1, device 4: "/sys/devices/platform/soc/20980000.usb/usb1/1-1/1-1.1"
May 11 21:48:33 raspberrypi mtp-probe: bus: 1, device: 4 was not an MTP device
May 11 21:48:34 raspberrypi kernel: [   86.852779] usbcore: registered new interface driver usbserial
May 11 21:48:34 raspberrypi kernel: [   86.853023] usbcore: registered new interface driver usbserial_generic
May 11 21:48:34 raspberrypi kernel: [   86.853196] usbserial: USB Serial support registered for generic
May 11 21:48:34 raspberrypi kernel: [   86.867455] usbcore: registered new interface driver pl2303
May 11 21:48:34 raspberrypi kernel: [   86.867639] usbserial: USB Serial support registered for pl2303
May 11 21:48:34 raspberrypi kernel: [   86.867815] pl2303 1-1.1:1.0: pl2303 converter detected
May 11 21:48:34 raspberrypi kernel: [   86.925572] usb 1-1.1: pl2303 converter now attached to ttyUSB0
```

This time I found what I was looking for. The last line of output shows the GPS receiver was attached to `ttyUSB0`.

## Testing GPS

Now that I knew where the GPS receiver was, I could continue along with the article and test that it was receiving coordinates.

```
$ stty -F /dev/ttyUSB0 4800
$ cat /dev/ttyUSB0
```

I could see the NMEA sentences containing the GPS data. Mine looked like:

```
$GPRMC,020256.000,A,4034.3172,N,08937.9087,W,4.65,0.91,080517,,,A*7C
```

The rest of the article worked like a charm. I cloned the [Git repo](https://github.com/mrichardson23/gps_experimentation) containing the Python script for reading and parsing the data and set them to run on boot. After that, I took a short drive around the neighborhood to see what data I collected.

## Working with the data

For now, it's easiest to pull the data off the Zero W and work with it on my main machine. Since I know where the data is located at, scp is the fastest way to get at it.

```
$ scp pi@192.168.1.123:/home/pi/gps_experimentation/*.txt ~/Routes/
```

NMEA sentences aren't exacly readable and I wanted to view the routes on a map. The Python script automatically creates a separate file with decimal coordinates that can be viewed in `gpsprune`. That didn't fit the final outcome I was looking for.

I'm a heavy Nextcloud user and rembered seeing the [GpxPod](https://apps.nextcloud.com/apps/gpxpod) app. It will search your Nextcloud files for GPS data files. Since GPX was the only format that didn't require additional depencencies, I looked into converting the NMEA sentences to GPX data.

I love writing command-line tools in PHP, so I wrote a [program](https://github.com/adamkammeyer/gphps) to parse through the raw GPS data and output a GPX file. I can tell the program to place the GPX file in my Nextcloud sync folder and it's instantly uploaded to my server. I created a "Routes" directory to keep things organized.

Once I was logged into my Nextcloud instance, I enabled GpxPod. Easily done by going to Apps > Tools and clicking the Enable button.

After GpxPod was enabled, I loaded the app and chose my `/Routes` folder from the Folder list. It scanned the folder and gave me a list of Routes to choose from to display on the embedded OpenStreetMap.

## Extras

The extra parts I ended up using were strictly for convenience and added after I had everything up and running. The 90 degree angled USB cables saved some much needed room. The on/off switch just made it easier to turn on the Zero when I needed it. Plugging in and unplugging the micro USB cable from the Zero was difficult around all the cables.

## Next Steps

With this initial setup, I can at least capture my route data and get it into a format to see on a map. I already have ideas for enhancements. While creating the GPX file, I'd like to determine the elevation for each coordinate point. I'm also contemplating incorporating the [Pimoroni Touch pHAT](https://www.adafruit.com/product/3472) to let me manually start/stop the Python script and start the GPX conversion. Another thought was using something like the [PaPiRus Zero ePaper/eInk pHAT](https://www.adafruit.com/product/3335) to output information (e.g., time, speed, etc.) live while capturing. However, that would probably mean creating some kind of mount for my bike to be able to see the screen. Ah, possibilities. :)
