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
KERNEL="4.1"

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
zzz
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

mkfs.vfat -F32 /dev/${TARGET}1
mkswap /dev/${TARGET}3
swapon /dev/${TARGET}3
zpool create $ZPOOL ${TARGET}2
zfs create $ZPOOL/$ROOTFS
zfs create $ZPOOL/$ROOTFS/$INSTALLFS
zfs umount -a
zfs set mountpoint=/ $ZPOOL/$ROOTFS/$INSTALLFS
zpool set bootfs=$ZPOOL/$ROOTFS/$INSTALLFS $ZPOOL
zpool export $ZPOOL
zpool import -R /mnt $ZPOOL
mkdir -p /mnt/{boot,dev,proc,run,sys}
mount /dev/{TARGET}1 /mnt/boot
mount --rbind /dev /mnt/dev
mount --rbind /proc /mnt/proc
mount --rbind /run /mnt/run
mount --rbind /sys /mnt/sys
xbps-install wget xz
wget "http://repo.voidlinux.eu/static/xbps-static-latest.x86_64-musl.tar.xz"
tar xf xbps-static-latest.x86_64-musl.tar.xz -C /mnt
/mnt/usr/bin/xbps-install -S --repository=http://repo.voidlinux.eu/current -r /mnt base-system
cp /etc/resolv.conf /mnt/etc/resolv.conf
cp /etc/wpa_supplicant/wpa_supplicant-$WIFIDEV.conf /mnt/etc/wpa_supplicant/wpa_supplicant-$WIFIDEV.conf
chroot /mnt /bin/bash

passwd root
chown root:root /
chmod 755 /
echo $HOSTNAME > /etc/hostname
xbps-install zfs efibootmgr curl unzip
(cd /boot; curl -O -J -L "http://sourceforge.net/projects/refind/files/latest/download?source=files" && unzip refind-bin-*.zip && ./refind-bin-*/install.sh)
printf '/dev/${TARGET}1\t/boot\tvfat\tdefaults\t\t0\t0\n' >> /mnt/etc/fstab
printf '/dev/${TARGET}3\tswap\tswap\tdefaults\t\t0\t0\n' >> /mnt/etc/fstab
printf 'hostonly=yes\n' >> /etc/dracut.conf
zpool set cachefile=/etc/zfs/zpool.cache $ZPOOL
echo "LANG=es_GB.UTF-8" > /etc/locale.conf
xbps-reconfigure -f glibc-locales
xbps-reconfigure -f linux${KERNEL}
echo "now add 'zfs=bootfs' to standard options of /boot/refind_linux.conf"
ln -s /etc/sv/dhcpcd /var/service/

useradd -m -s /usr/bin/zsh -G wheel,users,audio,video,cdrom,input joe
passwd joe
xbps-install zsh
echo "/usr/bin/zsh" >> /etc/shells
zfs snapshot $ZPOOL/$ROOTFS/$INSTALLFS@fresh-install
exit
exit
umount -R /mnt
```
