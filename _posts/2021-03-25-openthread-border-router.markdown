---
layout: post
title:  "OpenThread Border Router with Raspberry Pi"
date:   2021-03-25 08:56:46 -0400
categories: openthread rpi
---

# OpenThread

OpenThread is an opensource implementation of the [thread][thread-primer] — an IPv6-based networking protocol using the IEEE 802.15.4 physical layer (zigbee).  Its can be implemented on many simple devices and development tools from popular manufacturers like nordic and stm.


modify the file `/boot/config.txt`:

{% highlight bash %}
enable_uart=1
dtoverlay=pi3-disable-bt
{% endhighlight%}



[thread-primer]: https://openthread.io/guides/thread-primer
