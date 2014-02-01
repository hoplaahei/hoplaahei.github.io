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

First, choose the disk you want to install Arch on. Use `blkid` to see available devices. I like to get rid of remenants of old partitions on the chosen disk by running:

```
gdisk /dev/sdX # where X is the device number you selected
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

We will put LVM in its own partition; that way it can coexist with the EFI boot partition we made. As a side note, LVM can exist without a partition on the block level (e.g., as /dev/sdX rather than /dev/sdX2), but we won't concern ourselves with that. 

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

- choose 1, 2, and 3, then skip to '6) Install Base System'
- choose 7) through 12)
- if using an SSD with trim support (check with `hdparm -I /dev/sdX |grep TRIM`), then change /etc/fstab options to `defaults,noatime,discard` for each ext4 partition listed. Or edit /etc/lvm/lvm.conf to contain `issue_discards = 1`
- exit ais script, but choose not to reboot
- edit /etc/mkinitcpio.conf 'HOOKS=...' and stick `lvm2` in between `block` and `filesystems`
- save changes and load `./ais` again
- run option 12) again to build lvm2 support into the image
- do not manually run `mkinitcpio` (unless you `arch-chroot` into /mnt)

Now exit out ./ais once more, without rebooting, and run:

```
arch-chroot /mnt
pacman -S gummiboot
gummiboot install
```

Edit /boot/loader.conf:

```
default  arch
timeout  3 # how long you want menu to stay before booting
```

Make /boot/loader/entries/arch.conf:

```
title          Arch Linux
linux          /vmlinuz-linux
initrd         /initramfs-linux.img
options        root=/dev/mapper/VolGroup00-lvolroot rw
```
You now have a base Arch Linux installation you can boot into:

```
exit
umount /mnt/*
umount /mnt
reboot
```

Tip: Arch is rolling release and in my experience from time-to-time an update breaks the bootloader and locks you out the Arch installation. If this happens you can stick the Arch USB key in (which itself has a working UEFI bootloader) and it will try to boot into your Arch installation rather than the live environment by default. From there you can use [Arch Rollback Machine](https://wiki.archlinux.org/index.php/Arch_Rollback_Machine) and get an older working version of gummiboot. See: [what to do when things go wrong](#). 

## Basic configuration

The `ais` script from the install copied `aui` over to /root, so when you first login run `./aui`. This will help you the rest of the way in setting up the system. At the least, let it install an [AUR](https://aur.archlinux.org) helper such as `yaourt`. 

It can also automatically set up many great optimisations such as:

- `powerpill` for quicker downloading
- `bumblebee` for NVIDIA Optimus (graphics switching/power saving) technology
- `infinality` for smooth fonts
- `ZRAM` for using RAM for swap, thus reducing disk writes
- `readahead` for caching common used apps for quicker loading

Furthermore, it reminds you about other things you may need such as CUPS printing and openssh. 

## Power saving for laptops

... [coming soon]



