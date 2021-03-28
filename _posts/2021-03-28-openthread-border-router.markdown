---
layout: post
title:  "OpenThread Border Router with Raspberry Pi - Part 1"
date:   2021-03-28 08:56:46 -0400
categories: openthread rpi border router
---

# OpenThread

OpenThread is an opensource implementation of the [thread][thread-org] — an IPv6-based networking protocol using the IEEE 802.15.4 physical layer (zigbee).  Its can be implemented on many simple devices and development tools from popular manufacturers like nordic and stm.

## Overview

This configuration will use a Raspberry Pi as a border router -- connecting a OpenTread Network to an external Network.  The raspberry pi will be used to connect to any network via its built in WiFi or Ethernet ports.  Connected to it will be an RCP, Radio Co-Processor, which will be the physical interface to the Openthread layer.

This is largely an elaboration on the [how-to][ot-br-howto] openthread provides, but covering a few of the missing details and changes since its orignial publish (though unclear how long this will be remain accurate either)

For this specific version the RCP will be built ontop of the Nordic [nRF52840-DK], which will connect to the Raspberry Pi via USB.

[nRF52840-DK]: https://www.nordicsemi.com/Software-and-Tools/Development-Kits/nRF52840-DK

# Setup RPi

First setup the raspberry pi.  The following materials were used:

* Raspberry Pi 3B+
* 16GB uSD card
* [FTDI-Style (Serial<>USB) Cable][ftdi]

Download and install the raspberry pi image tool: <https://www.raspberrypi.org/software/>

**1 -** Insert The SD card into you computer.

![imager-screen-grab](/assets/images/rpi-imager.png)

**2 -** Click `CHOOSE OS` -> Select `Other` -> `Raspberry Pi OS Lite (32-bit)`

**3 -** Click `CHOOSE STORAGE` to select the SD card you inserted previously.

**4 -** Click `WRITE`

The system will now download and install the image on the SD card.

## Configure SD Card for FTDI

Once its complete modify the system's boot configuration to enable the serial port to work with the FTDI cable.  This way we can use the raspberry pi headless (without screen/keyboard) before we setup networking to be able to use SSH.

Mount the SD card you just imaged (specifically the boot partition).

modify the file `/boot/config.txt` and add the following lines to the end:


    enable_uart=1
    dtoverlay=pi3-disable-bt

After this is complete unmount the SD card and insert it into the raspberry pi.

# Connect the Pi

Reading the [pinout], connect the FTDI cable with the **RX (Pin 10)**, **TX (Pin 8)**, and **GND (Pin 6)** using to the **Green**, **White**, and **Black** wires, respectivly.

[pinout]: https://pinout.xyz/pinout/uart#

Attach the USB cable to a free USB port on your computer

## Screen

Screen is a simple program for the serial echo to the shell.  To install in ubuntu:

    sudo apt install screen

### Using Screen

The cable will be powered by the USB port and you should see a new serial device in your machines device tree.  Once you attach it the new device should be automatically loaded.  You can run `dmesg` to see what device it is:

    $ dmesg
    ...
    [707089.884761] usb 1-2: new full-speed USB device number 5 using xhci_hcd
    [707090.033809] usb 1-2: New USB device found, idVendor=067b, idProduct=2303, bcdDevice= 3.00
    [707090.033810] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [707090.033811] usb 1-2: Product: USB-Serial Controller
    [707090.033812] usb 1-2: Manufacturer: Prolific Technology Inc.
    [707090.035179] pl2303 1-2:1.0: pl2303 converter detected
    [707090.035845] usb 1-2: pl2303 converter now attached to ttyUSB0

Screen will require su permissions to use the device, and specifiy the standard speed of the serial port.

    sudo screen /dev/ttyUSB0 115200

This will start a screen terminial to interface with the Raspberry Pi.

# Boot Up The Pi

Attach a micro-USB cable to the pi and power it with a USB PS providing 5V at least 1A.  _Note: You may get undervoltage issues with if the power supplied is low on current or the cable used is too small a gauge_

You should see the LEDs light up and the system boot in the screen terminal window:

    ...
    [    5.045680] usb 1-1.1.2: new full-speed USB device number 4 using dwc_otg
    [    5.075112] systemd[1]: Set hostname to <raspberrypi>.
    [    5.191577] usb 1-1.1.2: config 1 has an invalid interface number: 2 but max is 1
    [    5.205123] usb 1-1.1.2: config 1 has no interface number 0
    [    5.229665] usb 1-1.1.2: New USB device found, idVendor=1915, idProduct=cafe, bcdDevice= 1.00
    [    5.244527] usb 1-1.1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [    5.257911] usb 1-1.1.2: Product: nRF528xx OpenThread Device
    [    5.266853] usb 1-1.1.2: Manufacturer: Nordic Semiconductor
    [    5.275609] usb 1-1.1.2: SerialNumber: EBC17A55B016
    [    5.315020] uart-pl011 3f201000.serial: no DMA platform data
    [    5.605432] usb 1-1.1.1: new high-speed USB device number 5 using dwc_otg
    [    5.755969] usb 1-1.1.1: New USB device found, idVendor=0424, idProduct=7800, bcdDevice= 3.00
    [    5.770257] usb 1-1.1.1: New USB device strings: Mfr=0, Product=0, SerialNumber=0
    [    6.014296] random: systemd: uninitialized urandom read (16 bytes read)
    [    6.042380] random: systemd: uninitialized urandom read (16 bytes read)
    [    6.052787] lan78xx 1-1.1.1:1.0 (unnamed net_device) (uninitialized): No External EEPROM. Setting MAC Speed
    [    6.053870] systemd[1]: Listening on Journal Socket.
    [    6.077464] libphy: lan78xx-mdiobus: probed
    [    6.088870] random: systemd: uninitialized urandom read (16 bytes read)
    [    6.099189] systemd[1]: Listening on Journal Audit Socket.
    [    6.114505] systemd[1]: Created slice system-serial\x2dgetty.slice.
    [    6.128616] systemd[1]: Listening on udev Control Socket.
    [    6.141165] systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
    [    6.160586] systemd[1]: Set up automount Arbitrary Executable File Formats File System Automount Point.
    [    6.187314] systemd[1]: Condition check resulted in Set Up Additional Binary Formats being skipped.
    [    6.208611] lan78xx 1-1.1.1:1.0 (unnamed net_device) (uninitialized): int urb period 64

    Raspbian GNU/Linux 10 raspberrypi ttyAMA0

    raspberrypi login: 

This will show everything has been configured properly up to this point.  You can login using `pi` / `raspberry` 


After everything comes up you will need to install a few things from the jump...specifically `vim` (or another perfereable text editor) and `git` (the version control software)

    sudo apt install vim git


Download the border router source:

    git clone https://github.com/openthread/ot-br-posix


# Build and Flash The RCP

Using a dev machine you need to build and flash the nRF52840-DK.  


## RCP Build for nrf52

Using a directory `projects` in the home folder for this work, anywhere can do the trick though.
    
    cd ~/projects
    git clone --recursive https://github.com/openthread/ot-nrf528xx.git
    cd ot-nrf528xx
    script/build nrf52840 USB_trans

Once Built you will need to convert the binary to hex:
    
    cd ~/projects/ot-nrf528xx/build/bin
    arm-none-eabi-objcopy -O ihex ot-rcp ot-rcp.hex


To program the board download and install the [nRF Command Line Tools][nrf-tools] 

[nrf-tools]: https://www.nordicsemi.com/Software-and-Tools/Development-Tools/nRF-Command-Line-Tools/Download

Once you download and install flash the nRF using the nrfprog tool

    nrfjprog -f nrf52 --chiperase --program ~/projects/ot-nrf528xx/build/bin/ot-rcp.hex --reset
    
optionally if you have more then one board attached its useful to pass a seial number as an arg here as well, e.g. `-s 683934234`

    nrfjprog -f nrf52 -s 683934234 --chiperase --program ~/projects/ot-nrf528xx/build/bin/ot-rcp.hex --reset


In the next part well cover putting all of these togeather and getting the network stood up.


[thread-primer]: https://openthread.io/guides/thread-primer
[thread-org]: https://www.threadgroup.org/What-is-Thread/Thread-Benefits
[ot-br-howto]: https://openthread.io/guides/border-router/build
[ftdi]: https://www.adafruit.com/product/954

