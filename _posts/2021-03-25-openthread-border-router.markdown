---
layout: post
title:  "OpenThread Border Router with Raspberry Pi"
date:   2021-03-25 08:56:46 -0400
categories: openthread rpi
---

# OpenThread

OpenThread is an opensource implementation of the [thread][thread-org] — an IPv6-based networking protocol using the IEEE 802.15.4 physical layer (zigbee).  Its can be implemented on many simple devices and development tools from popular manufacturers like nordic and stm.


modify the file `/boot/config.txt`:

{% highlight bash %}
enable_uart=1
dtoverlay=pi3-disable-bt
{% endhighlight%}


https://openthread.io/guides/border-router/build


    sudo apt install vim git



    git clone https://github.com/openthread/ot-br-posix


RCP Build for nrf52


    $ cd ~/projects
    $ git clone --recursive https://github.com/openthread/ot-nrf528xx.git
    $ cd ot-nrf528xx
    $ script/build nrf52840 USB_trans


    $ cd ~/projects/ot-nrf528xx/build/bin
    $ arm-none-eabi-objcopy -O ihex ot-rcp ot-rcp.hex


    nrfjprog -f nrf52 --chiperase --program ~/projects/ot-nrf528xx/build/bin/ot-rcp.hex --reset
    



https://openthread.io/platforms/co-processor/firmware



    nrfjprog -f nrf52 --chiperase --program ~/Downloads/ot-ncp-ftd-gae2b0194-nrf52840.hex --reset



[thread-primer]: https://openthread.io/guides/thread-primer
[thread-org]: https://www.threadgroup.org/What-is-Thread/Thread-Benefits
