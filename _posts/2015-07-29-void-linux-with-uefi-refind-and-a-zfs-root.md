---
layout: post
published: false
title: Void (Linux) with rEFInd (UEFI) and ZFS (root)
---

Void Linux is installable from any existing OS or live CD. Find a recent one that comes with ZFS tools (or can install the tools).

```
#!/usr/bin/env bash

set -euf -o pipefail

# user set variables
TARGET="/dev/sda"
ZPOOL="rpool"
ROOTFS="ROOT"
INSTALLFS="voidlinux_1"
EFISIZE=512
HOSTNAME="salient230"

# don't change these
DISKSIZE=$(blockdev --getsize64 $TARGET) 
SWAPSIZE=$(vmstat -s -S b | awk 'NR <=1 {print $1 * 1.5}')

# size of disk minus space needed for swap minus space needed for an EFI partition 
# EFI size converted to bytes for the sum
# answer converted to kilobytes (because gdisk doesn't accept bytes)
# don't change this
ROOTSIZE=$(( ( $DISKSIZE - ($SWAPSIZE + (( ($EFISIZE * 1024) * 1024 )) ) ) / 1024 ))

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

mkfs.vfat -F32 /dev/sda1
zpool create $ZPOOL ${TARGET}2
zfs create $ZPOOL/$ROOTFS
zfs create $ZPOOL/$ROOTFS/$INSTALLFS
zfs umount -a
zfs set mountpoint=/ $ZPOOL/$ROOTFS/$INSTALLFS
zpool set bootfs=$ZPOOL/$ROOTFS/$INSTALLFS $ZPOOL
zpool export $ZPOOL
zpool import -R /mnt $ZPOOL
mkdir -p /mnt/{boot/efi,dev,proc,run,sys}
mount /dev/sda1 /mnt/boot/efi
mount --rbind /dev /mnt/dev
mount --rbind /proc /mnt/proc
mount --rbind /run /mnt/run
mount --rbind /sys /mnt/sys
xbps-install wget xz
wget "http://repo.voidlinux.eu/static/xbps-static-latest.x86_64-musl.tar.xz"
tar xf xbps-static-latest.x86_64-musl.tar.xz -C /mnt
/mnt/usr/bin/xbps-install -S --repository=http://repo.voidlinux.eu/current -r /mnt base-system
cp /etc/resolv.conf /mnt/etc/resolv.conf
chroot /mnt /bin/bash
passwd root
chown root:root /
chmod 755 /
echo $HOSTNAME > /etc/hostname
xbps-install zfs

```




