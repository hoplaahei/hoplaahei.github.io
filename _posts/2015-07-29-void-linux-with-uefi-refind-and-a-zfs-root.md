---
layout: post
published: false
title: Void Linux with UEFI (rEFInd) and a ZFS root
---

Void Linux is installable from any existing OS or live CD. Find one that is fairly recent, and comes with ZFS tools (or allows you to install them).

```
TARGET=/dev/sda
WIFIDEV=wlp3s0
ESSID=PlusnetWireless94CBB7
WIFIPASS=9FB2A30C04

# securely erase our target disk
hdparm --user-master u --security-set-pass pAsSwOrD $TARGET
hdparm --user-master u --security-erase pAsSwOrD $TARGET

# bring up the wifi
cp /etc/wpa_supplicant/wpa_supplicant{,-$WIFIDEV}.conf
 wpa_passphrase $ESSID $WIFIPASS >> /etc/wpa_supplicant/wpa_supplicant-$WIFIDEV.conf
sv restart dhcpcd




