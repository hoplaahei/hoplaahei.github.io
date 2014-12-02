---
layout: post
published: false
title: TrueOS On Laptops
---

TrueOS is almost vanilla FreeBSD with a few tweaks and cli versions of most PC-BSD tools. It is designed for server installations, but I like to use it as a way to get a FreeBSD installation for my laptop up and running with a ZFS-on-root quickly. 

First thing to do when booted into TrueOS is get the wifi up. In `/etc/rc.conf` ammend:

```
ifconfig_wlan0="DHCP"
```

Change it to:

```
ifconfig_wlan0="WPA DHCP"
```

This simple change will invoke wpa_supplicant on boot. Now edit `/etc/wpa_supplicant.conf` from the cli with:

```
wpa_passphrase yourAP yourPass > /etc/wpa_supplicant.conf
```
Restarting the network service after this change is finicky (at least on my Centrino 6300), so I found a reboot was needed to get the wifi adapter up and connected to the AP.


