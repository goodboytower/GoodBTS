# GoodBTS

--- Apologies for the guide still being incomplete. I'm short on time, but I will continue to work on this until I see it as acceptable. Give it a month or so! ---

The most complete and up-to-date repository for [Nuand BladeRF](https://shrinkearn.com/ooNe) and [YateBTS](https://shrinkearn.com/1YS1) compatibility.

After working through the various issues of multiple sources to implement YateBTS with the BladeRFx40, I decided to create this repository in order to assist other creators with their own implementation in a much more thorough educational guide. I will do my best to avoid gaps in implementation, and add educational pieces where I feel confident in my knowledge. 

This educational guide was written by Michael Maune. A big shoutout to all those who helped and assisted with this project: Giovanni Licamelli, Mohammed Islam, David Miller, "rtucker at Nuand.com", and "Ioana Stanciu at forum.yate.ro".

DISCLAIMER: This guide is only intended for use in educational projects. Using this guide will create a functional Network-In-A-Box (NIB) for personal use. GoodBTS will NOT assist in creating an IMSI Catcher or similar man-in-the-middle functions. Unethical use of the hardware referred to in this guide is ILLEGAL and restricted by the FCC. Please refer to [FCC §97.313(a)](https://shrinkearn.com/1vLf) for further guidance on broadcasting in the RF spectrum. 

Thanks for putting up with the ads in the links! In return, I plan on keeping up with this project to answer questions unlike some BTS repositories out there. (Nothing wrong with them, but people move on to other projects sometimes. Hopefully I write this well enough to reduce the difficulties!)

### I. Introduction - Prerequisites

This project is implemented with a Raspberry Pi 3, a BladeRF x40, and SIM cards purchased separately (with ADM keys). 

#### Required Components
- [Raspberry Pi (Pi 3 Model B)](https://shrinkearn.com/7lBgo)
- SD Card (minimum 8 GB)
- Externals (Mouse, keyboard, HDMI card and monitor)
- [Nuand BladeRFx40](https://shrinkearn.com/rFSM1) (Radio Frequency Transceiver)
- Quad-band Antennas 

##### Optional Components
- [SIM cards with ADM Keys](https://shrinkearn.com/al3UG)
- [SIM card Reader/Writer](https://shrinkearn.com/2E9M)
- [SIM card cutter (if necessary)](https://shrinkearn.com/4dqqT)

### II. Raspberry Pi Installation
A preliminary component to our BTS is a Raspberry Pi to function as the base operating system. We installed a well-known Linux distribution known as Raspbian.
1. Download the .img image of Raspbian or Raspbian Lite. (Raspbian Lite only has a command line interface).
2. Format your SD card and use Win32 Disk Imager to install the Raspbian distribution into a partition on the SD card.
3. Insert the SD card into the Raspberry Pi.
4. Connect your monitor, mouse and keyboard into the Raspberry Pi, and then insert the power cable. WARNING: Do not connect power until all other externals have been connected, preferably with a protective shell covering the green board.
5. Give the Raspberry Pi ample time to boot. Wait until you are prompted to login. This may take up to five minutes. The default username and password are pi and raspberry respectively. 

### III. Yate/YateBTS Installation in Raspbian
Yate is an acronym for Yet Another Telephony Engine that provides for a free and open source communication engine. YateBTS is a software implementation of the Yate engine that allows us to configure our BTS. [A wiki to YateBTS installation is here](https://shrinkearn.com/9UMX)

First, install dependencies. These may already be on your operating system, or may not be necessary for Yate but are helpful to have around. 

```
sudo apt-get install git php5 bladerf libbladerf-dev libbladerf0 automake gcc g++ libusb-1.0-0 libusb-1.0-0-dbg libusb-1.0-0-dev cmake doxygen kdoc
```
> LibbladeRF is no longer required due to Yate providing BladeRF support.

Verify your BladeRF firmware is 1.6.1 and FPGA is 0.1.2. The bladeRF firmware doesn’t play well with Yate unless the right firmware and FPGA are used, and this is what works for me. This may need some trial and error. Yate overrides the entire BladeRF C library with its own on execution, so your mileage may vary on different version of Yate/YateBTS.

To download the FPGA and firmware, run these commands. Put these both in a place you’ll remember - you’ll be needing them later. The links below have each of the images directly on the Nuand website.

```
wget https://www.nuand.com/fpga/v0.1.2/hostedx40.rbf //BladeRF FPGA 0.1.2
wget https://www.nuand.com/fx3/bladeRF_fw_v1.6.1.img //BladeRF Firmware 1.6.1
```
----- [Older versions of BladeRF firmware](https://shrinkearn.com/wmDGr)

----- [Older versions of BladeRF FPGAs](https://shrinkearn.com/DDxcX)

Next, download the tar of Yate and YateBTS. Yate's website has multiple versions of Yate/YateBTS.

```
wget http://yate.null.ro/tarballs/yatebts5/yate-bts-5.0.0-1.tar.gz // YateBTS V 5.5.1
wget http://voip.null.ro/tarballs/yate5/yate-5.5.0-1.tar.gz //Yate V 5.5.0
```
----- [Older versions of Yate](https://shrinkearn.com/Tto4N)

----- [Older versions of YateBTS](https://shrinkearn.com/PtC0)

> NOTE: The hardest challenge I faced was figuring out the right versions between the BladeRF FPGA, firmware, YateBTS and Yate versions. If you know the most recent compatible versions of each of these, I’d love to know the answer!

> WARNING: If you have more than one instance of yate-config, it would be best to remove all except the one you need. You can verify with ``` which -a yate-config ``` to find their location. Run ```make uninstall``` to remove it from inside that directory.

To install, first you will run the script that checks your dependencies.
```
root@raspberrypi:/home/pi/yate# ./autogen.sh
root@raspberrypi:/home/p/yatei# ./configure
```
Then, you will begin the installation of Yate. This process is identical to run in your YateBTS directory.

>Take note if these install correctly or not, for troubleshooting purposes. You may need to apply a gcc patch to the YateBTS directory, more on that further down.

```
root@raspberrypi:/home/pi/yate# make -j4
root@raspberrypi:/home/pi/yate# sudo make install
root@raspberrypi:/home/pi/yate# sudo ldconfig
root@raspberrypi:/home/pi/yate# cd ../
```

### Interacting with the BladeRF

Next, we configure the BladeRF. It's important to note that the bladeRF defaults to pull power from the USB cable, so be sure to find a separate power cord for the BladeRF. When connecting, if you receive an error suggesting there are no registered devices, that means you are already logged into the Pi in a different terminal or as a non-root user. [The wiki to the BladeRF basic device operation is here](https://shrinkearn.com/Gxp9)

To initialize and enter the BladeRF CLI, run this command
```
root@raspberrypi:/home/pi# bladeRF-cli -i
```

This will place you into the command line of the bladeRF. From here, you can start, update, and configure the bladeRF for testing purposes. If you have a handy spectrum analyzer, you can broadcast with dev/urandom. More on that later

You can check the image version that is installed with the following command. Note the bus and address portions of the output:
```
bladeRF> version
```

If you don't need to load a different firmware version, skip this step about flashing the images.

Now we need to flash the firmware image. Inside the BladeRF CLI, locate your firmware image and simply run:
``` 
bladeRF> load fx3 bladeRF_fw_v1.6.1.img
// or
bladeRF> <bus> <address> <path to firmware image>
```

From inside the BladeRF CLI, use the ```help``` page inside the CLI to calibrate your BladeRF. To verify your changes, use the ```print``` command to view your settings. 

[The link to the general BladeRF wiki is here](https://shrinkearn.com/78br)

### Web GUI for YateBTS

If not already installed, install apache web server, please do so. This will be necessary so we can reach the hosted web files that communicate with the physical Yate & YateBTS files. Note that we originally tried using the web interface, but had many issues with certain configuration files not installing in the correct place, and thus the web GUI couldn’t find where to write changes. So we had a healthy mix of configuring with the WebGUI and manually. You can configure these files completely manually as listed below, but rather use the WebGUI as a template to make the files with the below information to double check the parameters.

This next set of steps isn't critical, but it can be helpful if you want to get out of the command line. You'll need to have an Apache web server ready in order to reach your local config files. I had a healthy mix of configuring with the Web GUI and command line, but I'll do my best to explain both methods.

#### Install Apache

The recommended location to install is /usr/local/share/yate/nib/web for your Network In a Box (NIB) configuration.
```
root@raspberrypi:/home/pi# apt-get install apache2
root@raspberrypi:/home/pi# cd /var/www/html/
root@raspberrypi:/home/pi/var/www/html# ln -s /usr/local/share/yate/nib/web nib
root@raspberrypi:/home/pi# chmod a+w -r /usr/local/etc/yate
```

Now you should have your graphical interface installed on the web server. Go to your browser and access the server via ```localhost/nib``` or ```127.0.0.1/*name-of-directory*```
