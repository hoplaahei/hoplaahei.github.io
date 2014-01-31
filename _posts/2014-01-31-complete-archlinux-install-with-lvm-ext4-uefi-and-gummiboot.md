---
layout: post
published: true
---

Note: This guide requires knowledge of basic Linux Console commands. Please backup all data before attempting to follow it.

## Why UEFI?

Uefi seems complicated to setup, but once you know how, it is arguably simpler than the old MBR way. It also boots somewhat quicker on most systems. 

## Why gummiboot?

I've opted for gummiboot over Grub because, as of the time of writing, the pacman update installs a buggy Grub version that prevents the user making modifications to the bootloader. Fear not, gummiboot is very simple to use, and gets the job done. 

## Download the iso

Choose your nearest [mirror](https://www.archlinux.org/download/) to get the dual ISO in the fastest possible time.

## Copy ISO to USB pen

In the past, extra steps were needed to make the ISOs UEFI bootable, but now a `dd` command will suffice:

```bash
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdX && sync # where X is your device number
```

If unsure which `sdX` device to copy the ISO too, run `dmesg | tail` just after plugging it in and look at the last few lines.

## Modify BIOS to make computer UEFI bootable

Reboot your computer and immediately press the BIOS key (usually F1 or Escape). Here you should make sure under the 'Boot' section that Uefi boot is enabled. 

Also set the boot order in the BIOS to boot from USB, or find the key to load the boot menu when you reboot. Select your USB device, then the UEFI Arch entry (if it pops up). 

## Prepare the live environment

When logged in, type `wifi-menu` to get a list of wifi networks. Select your router and enter the security key (usually found on a sticker below the router). If your computer is plugged in directly to the wall then use `ifconfig` to list the correct ethernet device, then run `dhcpcd ethX`. 

Once the internet connection is established we can update the live system with the latest packages (before we create the partitions that Arch will reside on).

```
pacman -Syu
```

This update will take some time depending on the speed of your connection. Now let's download a script by helmuthdu to make installing Arch easy:

```
pacman -S git
git clone git://github.com/helmuthdu/aui
```
This script lets you configure partitions, but it doesn't support LVM (at the time of writing), so don't run it yet. 

First, choose the disk you want to install Arch on. I like to get rid of remenants of old partitions on the disk by running:

```
gdisk /dev/sdX
```
... and pressing `o` to create `an empty GUID partition table`. Then `w` to save and write the changes.

## Create the EFI boot partition

Make a 512M EFI (ee00 code) boot partition:

```bash
gdisk /dev/sdX # if you are not already running it
```

Now press `n` and choose `1` to select 1st partition. Leave start sector at default by pressing `Enter`, but change end sector to `+512M` (it is important to keep it at this size). For the type enter the code `ef00` (EFI). Remember to write the changes with `w`.

## Creating and mounting the LVM volumes

Creating LVM volumes seems abstract and complex at first compared to traditional partitioning of Linux, but really it just has a few more layers of complexity to remember. 

It may seem confusing to learn that LVM does not necessarily need a partition, as it can work on the block level. However, we WILL put it in its own partition; that way it can coexist with the UEFI boot partition we made. 

Our LVM partition uses the remaining space on the drive. The steps are:

- create a primary LVM partition (code 8e00 in gdisk)
- make the device LVM ready (pvcreate)
- create a volume group to contain the volumes (vgcreate)
- add volumes (lvcreate)

It is no harder than those four steps. The volume creation in the last step is like creating traditional partitions e.g., /, /home, and swap.

To create the LVM partition:

```
gdisk /dev/sdX
```
Press `n` and `Enter` to make partition 2. `Enter` again for default start block, and `Enter` once more for default end block. `8e00` to create type LVM and then `w` to write the changes. 

Now create the LVM hierarchy:

```bash
pvcreate /dev/sdX2 # activate
lvcreate /deb/sdX2
lvcreate -L 15G VolGroup00 -n lvolroot
lvcreate -C y -L 10G VolGroup00 -n lvolswap
lvcreate -l +100%FREE VolGroup00 -n lvolhome # use remaining space
```

Format the partitions:

```
mkfs.ext4 /dev/VolGroup00/lvolroot
mkfs.ext4 /dev/VolGroup00/lvolhome
mkswap /dev/VolGroup00/lvolswap
mkfs.vfat -F32 /dev/sdX1
```

And mount the partitions inside the live environment:

```
mount /dev/VolGroup00/lvolroot /mnt
mkdir /mnt/home
mount /dev/VolGroup00/lvolhome /mnt/home
mkdir -p /mnt/boot
mount /dev/sdX1 /mnt/boot
swapon /dev/VolGroup00/lvolswap
```

The only thing that differs here from what you may see in other UEFI install guides is that we mount the efi partition to `/boot` instead of `/boot/efi`. Why don't we want to use /boot/efi? Well, as the Arch Wiki says, you would need to run `gummiboot --path=$esp update` after each package update, and copy over the kernel and initramfs manually. Why bother doing that when we don't need to? Stick to using /boot.

## Installing Arch

```
cd aui && .ais
```

- choose first four options then skip to '6) Install Base System'