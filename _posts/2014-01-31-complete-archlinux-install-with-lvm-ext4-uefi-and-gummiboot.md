---
layout: post
published: true
---

Note: This guide requires knowledge of basic Linux Console commands. 

Uefi seems complicated to setup, but once you know how, it is arguably simpler than the old MBR way. It also boots somewhat quicker on most systems. 

I've opted for gummiboot over Grub because, at the time of writing, a pacman update installs a buggy Grub version that prevents the user making modifications to the bootloader. Fear not, gummiboot is very simple to use, and gets the job done. 

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

Also set the boot order in the BIOS to boot from USB, or find the key to load the boot menu when you reboot. Select the Uefi Arch entry and bootup. 

## Prepare the live environment

Type `wifi-menu` to get a list of wifi-networks, else follow Arch networking guides for setting up Ethernet. 

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
... and pressing `o` to create `an empty GUID partition table`. Now press `w` to save and write the changes.

## Create the EFI boot partition

Make a 512M EFI (ee00 code) boot partition:

```
gdisk /dev/sdX
```

Now press `n` and choose `1`. Leave start sector at default by pressing `Enter`, but change end sector to `+512M`. For the type enter the code `ef00` (EFI). Remember to write the changes with `w`.

## Create the LVM volumes

Creating LVM volumes seems abstract and complex at first compared to traditional partitioning of Linux, but it is really very simple. There are three steps:

- make the physical device LVM ready (pvcreate)
- create a volume group to contain the volumes (vgcreate)
- add volumes (lvcreate)

It is no harder than those three steps. The volume creation in the last step is like creating partitions e.g., /, /home, swap.


