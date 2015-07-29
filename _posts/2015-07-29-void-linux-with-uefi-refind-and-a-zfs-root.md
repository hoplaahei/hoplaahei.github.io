---
layout: post
published: false
title: Void (Linux) with rEFInd (UEFI) and ZFS (root)
---

Void Linux is installable from any existing OS or live CD. Find a recent one that comes with ZFS tools (or can install the tools).

```
#!/bin/bash

TARGET=/dev/sda
EFISIZE=512 # in MB
##DISKSIZE=`expr $(blockdev --getsize64 $TARGET) / 1024` # in bytes
DISKSIZE=$(awk -v dev="${TARGET}$" '$0 ~ dev {print $3}' /proc/partitions) # in KB
# make swap size 1.5 times system RAM
##SWAPSIZE=`expr $(vmstat -s -S b) / 1024` # in bytes
SWAPSIZE=$(awk 'NR <=1 {print $2 * 1.5}' /proc/meminfo) # in KB
# subtract swap space and EFI from size of disk to get root size
ROOTSIZE=`expr ($(DISKSIZE) - $(SWAPSIZE)) - $(expr $EFISIZE * 1024)` # in KB

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

# partition disks
xbps-install gptfdisk
gdisk $TARGET <<EOF
n


+512M
ef00
n


+$ROOTSIZE

n



8200
w
EOF
```




