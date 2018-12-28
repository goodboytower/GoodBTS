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

#### Configure ybts.conf
To use the web interface, go to the BTS Configuration tab and select the GSM tab. This is where you will enter the values for your tower to broadcast, which include frequencies, tower name, band, and power. Fill out the required (depending on yate version click “Advanced” to see the remaining fields) fields and hit Submit.

To ensure you put it into the right location, telnet into Yate with ```telnet localhost 5038``` while it is running and enter the command ```status engine```. This will tell you where Yate is pulling its various config files from.

Sample ybts.conf configuration:
```
Radio.Band=900
Radio.C0=3650
Identity.MCC=001
Identity.MNC=01
Identity.ShortName=DoesntMatter
Radio.PowerManager.MaxAttenDB=40
Radio.PowerManager.MinAttenDB=35
```
> Note: You can manually edit this file in ```/usr/local/etc/yate/ybts.conf```. If encountering a missing ybts.conf file error, it may need to be moved to the /usr/local/etc/yate folder for the web GUI to find the file to write to. If so, use this command from the directory: ```mv /your/directory/ybts.conf /usr/local/etc/yate``` 

**This next part is much easier to do in the Web GUI**

In the Web GUI, go the the Subscribers tab. There is a link to write SIMs or Manage SIMS. Here you will need to configure the regular expression (REGEXP). This creates a whitelist that allows a specific set of users to connect (based on SIM IMSI) to connect to the BTS. **ONLY USE AN IMSI THAT YOU ARE THE OWNER OF**. Me in particular, I ordered 10 SIMs each with their own IMSI. For ease and versatility, we configured the REGEXP to accept my array of 10 SIMs, but yours will be different. This is the first 14 numbers of all of the SIMs I had, and because they are sequenced the IMSI per card changed only 1 per card. The last section in brackets accepts all of our cards.

```^12340000005678[0-9]$```

This will accept all of our SIMs once programmed. You can avoid this as well by simply adding subscribers via the web interface manually, but do not do both. 
Your tower should now be fully configured and ready for operation. It will be able to broadcast at your set frequencies. 

### SIM Programming
For me, the web GUI for YateBTS always failed to program the SIMs, so I resorted to manually using pySim to program the SIM cards. 

To start, you’ll need to properly place a SIM into the SIM card reader. You may need to use the SIM cutter depending on the phone you intend on using. Once fitted, plug the SIM reader/writer into the Pi. We’ll need to download pySim to program the SIMs.

```root@raspberrypi:/home/pi# apt-get install pysim```

From here, grab the information that should be on hand for you with your SIM cards. You’ll need to find or decide on the information you'll put on the IMSI, like card type, country code, mobile country code, mobile network code, ADM pin, ICCID, KI, OPC, and a made up name for that SIM. Follow the following format to program the SIMs.

>WARNING: If you incorrectly program a SIM with the wrong ADM key, you will brick the SIM card after 3-4 unsuccessful tries. Command line works best for this section.

```
root@raspberrypi:/home/pi# cd pysim
root@raspberrypi:/home/pi/pysim# ./pySim-prog.py -p0 -t [card type] -a [ADM Pin] -x [mobile country code] -y [mobile network code] -i [IMSI] -s [iccid] -k [ki] -o [opc] -n [whatever name]
```

Here is an example of my command:
```
root@raspberrypi:/home/pi/pysim# ./pySim-prog.py -p0 -t sysmoUSIM-SJS1 -a 49681225 -x 001 -y 01 -i 123400000056781 -s 1234511000000678901 -k E848CE041B42E17012B1F94B545CE623 -o DA8FBF229BAEA1D428472D42538067A3 -n myname
```

### Begin Tower Broadcast
To start broadcasting, simply type the following as root after configuring the BladeRF:
```
yate -s
```
A full list of flags for the ```yate``` command [can be found here](https://shrinkearn.com/scGvp)

To see if the interface is actually up, you can telnet into the bladeRF:
```
root@raspberrypi:/home/pi# telnet localhost 5038
> javascript load nib.js
> nib list registered
```

The load nib.js command will load the Network-In-A-Box script that allows you to check registered devices (along with other functions), and the ```nib list registered``` command is an example that will list the registered and associated devices that are online (subscribers). 

If everything has gone well, wait about 5-10 minutes and you should receive a welcome message on your phone from "Eliza" **only** the first time it connects to the BTS. If not, disable LTE and enable Roaming mode on the phone. This is where the MCC and MNC come into play - your phone should see a tower that begins with your MCC 3-digit number that you assigned. If not, try a different usable frequency or check what frequencies your phone uses. Yate should assign you a default short number. This takes about 2-5 minutes to register the devices and for the built in YateBTS’s Eliza to assign a number to the device. Now, try communicating with one another on that network through SMS or voice.
