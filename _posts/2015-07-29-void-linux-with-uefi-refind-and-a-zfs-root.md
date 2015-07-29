---
layout: post
published: false
title: Void (Linux) with rEFInd (UEFI) and ZFS (root)
---

Void Linux is installable from any existing OS or live CD. Find a recent one that comes with ZFS tools (or can install the tools).

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
```




