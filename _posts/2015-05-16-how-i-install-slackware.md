---
layout: post
published: false
title: How I install Slackware
---


# Preparation

Disclaimer: these commands will wipe your disk. The commands use the form `sdX`, where you need to replace the `sdX` with e.g., `sda`, and where 'a' is usually the first disk (but double check with `fdisk` or `gdisk` to make sure). Google if you don't understand how to use these tools. 

## Get Slackware ISO onto a USB pen

See the [torrents](http://www.slackware.com/getslack/torrents.php) page. I chose [Slackware 14.1 64-bit](http://www.slackware.com/torrents/slackware64-14.1-install-dvd.torrent) from the bottom of the page. At the time of writing it is the stable edition.

Once downloaded, use a terminal to put the ISO on a USB stick:

```
dd if=slackware-iso-you-downloaded.iso of=/dev/sdX bs=1M && sync
```

## SSD memory cell clearing

The tl;dr of SSD memory cell clearing is that it could potentially give back the same write speeds the SSD had from the factory by running these commands:

```
echo -n "mem" > /sys/power/state # suspend to RAM to unfreeze security
hdparm --user-master u --security-set-pass PasSWorD /dev/sdX
hdparm --user-master u --security-erase PasSWorD /dev/sdX
```

The `ArchWiki` has a good [article](https://wiki.archlinux.org/index.php/SSD_memory_cell_clearing) on this, so I'm not going to reiterate the steps in detail.

## Partitioning

The SlackDocs wiki has an [install](http://docs.slackware.com/slackware:install) guide for standard installation. Some modern computers force the use of UEFI boot with a GPT partitioning scheme, and if you require/want this then see the UEFI [README](http://slackware.mirrorcatalogs.com/slackware64-14.1/README_UEFI.TXT).

Afer partitioning, simply run:

```
setup
```

If following the steps in my guide, then you are using a USB installation, so when prompted for the installation media choose USB, and the installer will detect the install packages correctly. EFI and swap partitions should get detected and formatted automatically. Follow the `SlackDocs` on [installation](http://docs.slackware.com/slackware:install) for further advice, but the installer is usually clever enough in figuring out what you want.

If following the partitioning scheme in this guide, then skip to the next section. If, however, you followed the links in SlackDocs for `LVM` (Logical Volume Manager) installation, then follow these additional steps before rebooting after the install:

```
mount -o bind /dev /mnt/dev
mount -o bind /proc /mnt/proc
mount -o bind /sys /mnt/sys
chroot /mnt
$( /usr/share/mkinitrd/mkinitrd_command_generator.sh -r )
exit
```

And modify `/boot/efi/EFI/Slackware/elilo.conf` for `UEFI` systems, or `/etc/lilo.conf` for `BIOS` systems to use the LVM volumes. Change the append line to:

```
append="root=/dev/yourVG/yourLV vga=normal ro"
```

Replace 'yourVG' with whatever you named the `Volume Group` and 'yourLV' with the `Logical Volume` you created.  

Now run:

```
eliloconfig # or liloconfig if you use lilo
```

And reboot.

## When the installation is done

Now is a good time to make a nice clean image of your installation to revert back to if things go wrong. This will save the hassle of having to go through the whole partitioning and setup procedure again. This image is also useful for deploying Slackware to other computers with similar sized disks (but remember to change `[e]ilo` and `fstab` entries). 

I don't recommend adding anything else to the backup image, except perhaps a basic configuration from the [Slackware Beginners Guide](docs.slackware.com/slackware:beginners_guide). Otherwise, you might not remember what was edited by the time it comes to actually needing the image. Save confusing yourself.

There is no absolute, universally agreed on method for backing up. Assuming you have a large enough spare drive around, I recommend the tried and true `dd` to a compressed image file for the first time backup. I'll explain this method now. 

Do not leave the system that needs backing up running unless the underlying filesystem supports freezing (e.g., XFS). Instead, reboot into a live environment such as the Slackware Install ISO.

I'm using XFS filesystem, which allows freezing the mounted disk, so I don't bother rebooting into a live environment. But to avoid any problems with the likes of PID files of running processes in the frozen image, I also switch to single-user mode to shut off as much as I can. To do so, run:

```
/sbin/init 1
```

That way a minimal number of things are running when it comes to entering the freeze command on the console.

Also, make sure the backup goes to a spare disk with at least nearly as much space as the full size of the disk that needs backing up, otherwise disk space might run out. 

Steps:

```
xfs_freeze -f /
dd if=/dev/sda | gzip > /path/to/backup/disc/location/ssd_backup.gz
```

## Learn to use Slackbuilds

[Install](http://www.sbopkg.org/downloads.php) `sbopkg` and sync with the remote repository by running:

```
sbopkg -r
```

Now follow this guide to [manage queue files easily](http://slackblogs.blogspot.co.uk/2014/01/managing-sbo-dependencies-easily.html).

## Bastardize your installation with multilib

Adding 32-bit support to any Linux system currently draws in an extra layer of complexity that we could all do without, but it's a necessary evil to get some applications running. For instance, many wine apps need 32-bit support, and I prefer the near native speed for games it gives me (no my laptop does not support [VGA passthrough](https://wiki.debian.org/VGAPassthrough) with KVM).

Fortunately, [slackpkg+](http://slakfinder.org/slackpkg+.html) makes converting Slackware to multilib easy:

```
installpkg slackpkg+-[downloaded-version]-noarch-2mt.txz
/usr/doc/slackpkg+-*/setupmultilib.sh
```

From the slackpkg+ documentation:

> Periodically you should run "slackpkg install multilib"
> after run "slackpkg upgrade-all"

## Support for nVidia Optimus

Someone made a one-liner to install it. See the [docs](http://docs.slackware.com/howtos:hardware:nvidia_optimus}. 
