# GoodBTS

--- Apologies for the guide still being incomplete. I'm short on time, but I will continue to work on this until I see it as acceptable. Give it a month or so! ---

The most complete and up-to-date repository for BladeRF and YateBTS compatibility.

After working through the various issues of multiple sources to implement YateBTS with the BladeRFx40, I decided to create this repository in order to assist other creators with their own implementation in a much more thorough educational guide. I will do my best to avoid gaps in implementation, and add educational pieces where I feel confident in my knowledge.

A big shoutout to all those who helped and assisted with this project. Giovanni Licamelli, Mohammed Islam, David Miller, "rtucker at Nuand.com", and "Ioana Stanciu at forum.yate.ro".

### I. Introduction - Prerequisites

This project is implemented with a Raspberry Pi 3, a BladeRF x40, and SIM cards purchased separately (with ADM keys). 

#### Required Components
- SD Card (minimum 8 GB)
- Externals (Mouse, keyboard, HDMI card and monitor)
- bladeRF (Radio Frequency Transceiver)
- Raspberry Pi (Pi 3 Model B)
- Quad-band Antennas (Recommended)
- Ethernet connection/cable (for Pi)

##### Optional Components
- SIM cards with ADM Keys
- SIM card Reader/Writer
- SIM card cutter

### II. Raspberry Pi Installation
A preliminary component to our BTS is a Raspberry Pi to function as the base operating system. We installed a well-known Linux distribution known as Raspbian.
1. Download the .img image of Raspbian or Raspbian Lite. Raspbian Lite will only allow a command line interface only.
2. Format your SD card and use Win32 Disk Imager to install the Raspbian distribution into a partition on the SD card.
3. Insert the SD card into the Raspberry Pi.
4. Connect your monitor, mouse and keyboard into the Raspberry Pi, and then insert the power cable. WARNING: Do not connect power until all other externals have been connected, preferably with a protective shell covering the green board.
5. Give the Raspberry Pi ample time to boot. Wait until you are prompted to login. This may take up to five minutes. The default username and password are pi and raspberry respectively. 

#### Yate/YateBTS Installation in Raspbian
> Yate is an acronym for Yet Another Telephony Engine that provides for a free and open source communication engine. YateBTS is a software implementation of the Yate engine that allows us to configure our BTS.

First, install dependencies. These may already be on your operating system, or may not be necessary for Yate but are helpful to have around. 
> LibbladeRF is no longer required due to Yate providing BladeRF support.
```
sudo apt-get install git php5 bladerf libbladerf-dev libbladerf0 automake gcc g++ libusb-1.0-0 libusb-1.0-0-dbg libusb-1.0-0-dev cmake doxygen kdoc
```

Verify your BladeRF firmware is 1.6.1 and FPGA is 0.1.2. The bladeRF firmware doesn’t play well with Yate unless the right firmware and FPGA are used, and this is what works for me. This may need some trial and error. Yate overrides the entire BladeRF C library with its own on execution, so your mileage may vary on different version of Yate/YateBTS.

Older versions of BladeRF firmware: https://www.nuand.com/fx3_images/

Older versions of BladeRF FPGAs: https://www.nuand.com/fpga_images/ 

```
wget https://www.nuand.com/fpga/v0.1.2/hostedx40.rbf //BladeRF FPGA 0.1.2
wget https://www.nuand.com/fx3/bladeRF_fw_v1.6.1.img //BladeRF Firmware 1.6.1
```

Put these in a place you’ll remember - we’ll be needing them later. 
From the root folder of your Pi, download the repository of EvilBTS

Older versions of Yate: http://old.yate.ro/pmwiki/index.php?n=Main.Download

Older versions of YateBTS: https://wiki.yatebts.com/index.php/Old_versions 

```
wget http://yate.null.ro/tarballs/yatebts5/yate-bts-5.0.0-1.tar.gz // YateBTS V 5.5.1
wget http://voip.null.ro/tarballs/yate5/yate-5.5.0-1.tar.gz //Yate V 5.5.0
```

> NOTE: The hardest challenge I faced was figuring out the right versions between the BladeRF FPGA, firmware, YateBTS and Yate versions. If you know the most recent compatible versions of each of these, I’d love to know the answer!
