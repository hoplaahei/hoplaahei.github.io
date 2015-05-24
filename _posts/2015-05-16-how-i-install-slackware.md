---
layout: post
published: true
title: How I install Slackware
---


# Preparation

Disclaimer: these commands will wipe your disk. The commands use the form `sdX`, where you need to replace the `sdX` with e.g., `sda`, and where 'a' is usually the first disk (but double check with `fdisk` or `gdisk` to make sure). Google if you don't understand how to use these tools. 

## Get Slackware ISO onto a USB pen

See the [torrents](http://www.slackware.com/getslack/torrents.php) page. I chose [Slackware 14.1 64-bit](http://www.slackware.com/torrents/slackware64-14.1-install-dvd.torrent) from the bottom of the page. At the time of writing it is the stable edition.

Even though the ISO is for DVDs, it works just as well on a USB stick (replace sdX with the USB stick device):

```
dd if=slackware-DVD-iso-you-downloaded.iso of=/dev/sdX bs=1M && sync
```

## SSD memory cell clearing

This step is not at all essential to installing Slackware, but it is something I always do out of habit. The tl;dr of SSD memory cell clearing is that it could reset any slow down in SSD write speeds by returning it to factory defaults:

```
echo -n "mem" > /sys/power/state # suspend to RAM to unfreeze security
hdparm --user-master u --security-set-pass PasSWorD /dev/sdX
hdparm --user-master u --security-erase PasSWorD /dev/sdX
```

The `ArchWiki` has a good [article](https://wiki.archlinux.org/index.php/SSD_memory_cell_clearing) on this, so I'm not going to reiterate the steps in detail.

## Partitioning

The SlackDocs wiki has an [install](http://docs.slackware.com/slackware:install) guide for standard installation. Some modern computers force the use of UEFI boot with a GPT partitioning scheme, and if you require/want this then see the UEFI [README](http://slackware.mirrorcatalogs.com/slackware64-14.1/README_UEFI.TXT).

After partitioning, simply run:

```
setup
```

If following the steps in my guide, then you are using a USB installation, so when prompted for the installation media choose USB, and the installer will detect the install packages correctly. EFI and swap partitions should get detected and formatted automatically. Follow the `SlackDocs` on [installation](http://docs.slackware.com/slackware:install) for further advice, but the installer is usually clever enough in figuring out what you want.

If following the partitioning scheme in the SlackDocs wiki, then skip to the next section. If, however, you followed the links in SlackDocs for `LVM` (Logical Volume Manager) installation, then follow these additional steps before rebooting after the install:

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

## Get correct keys on the keyboard

After logging in it is useful to start `X` with `startx` command to run the default window manager selected in installation e.g., `windowmaker`, but the keys might not be right on the keyboard. For a quick fix, run from the terminal e.g.,

```
setxkbmap gb # for a British keyboard
```

This will only work for the current session, but we will set it permanently later. 

## Wireless networking

Enable `NetworkManager` at boot:

```
chmod +x /etc/rc.d/rc.networkmanager
```

Start NetworkManager:

```
/etc/rc.d/rc.networkmanager start
```

From the terminal list the available access points:

```
nmcli dev wifi
```

Connect to one (start this line with a space to prevent the wifi password getting stored in the history file of the shell)

```
 nmcli dev wifi connect YourAccessPoint password YoUrPaSs
```

## Follow the beginners guide

Do all the steps in the [beginners guide](http://docs.slackware.com/slackware:beginners_guide) and also follow the link to [localisation](http://docs.slackware.com/slackware:localization), as you can do useful things like get the '£' sign working permanently on a UK keyboard in `X`. 

I advise to read the comments of `/etc/profile.d/lang.sh` when setting a locale and not enable UTF-8 unless absolutely necessary. I experienced strange behaviour with some applications and UTF-8, e.g., `xpdf` failing to load some PDFs. If, however, UTF-8 is essential to your system, then you can still get the problem apps to load by overriding the locale per-application with `C` locale like this:

```
LANG=C xpdf
```

## Using a generic kernel on UEFI systems

There are some additional steps to load the new generic kernel on UEFI systems using `elilo`. Copy the `initrd` and `vmlinuz` from that kernel to `/boot/efi/EFI/Slackware`, e.g., 

```
cp /boot/vmlinuz-generic-3.10.17 /boot/efi/EFI/Slackware/
cp /boot/initrd.gz /boot/efi/EFI/Slackware/
```

And edit `/boot/efi/EFI/Slackware/elilo.conf` as opposed to `/boot/lilo.conf`. The syntax is mostly the same as in the beginners guide example, but the `/boot/` part of the paths need removing, e.g.,:

```
image = vmlinuz-generic-3.10.17
        initrd = initrd.gz
        root = /dev/sdb3
        label = 3.10.17
        read-only
        append="vga=normal ro"
```

## Enable resume from hibernation

In `/etc/lilo.conf` or `/boot/efi/EFI/Slackware/elilo.conf` add this somewhere in the double quotes (") of the `append=` line:

```
resume=/dev/sdX # where X is the swap partition
```

## Learn to use Slackbuilds

[Install](http://www.sbopkg.org/downloads.php) `sbopkg` and sync with the remote repository by running:

```
sbopkg -r
```

Now follow this guide to [manage queue files easily](http://slackblogs.blogspot.co.uk/2014/01/managing-sbo-dependencies-easily.html).

## Bastardise your installation with multilib

Adding 32-bit support to any Linux system currently draws in an extra layer of complexity that we could all do without, but it's a necessary evil to get some applications running. For instance, many wine apps need 32-bit support, and I prefer the near native speed for games it gives me (no my laptop does not support [VGA passthrough](https://wiki.debian.org/VGAPassthrough) with KVM).

`phenixia2003` has an excellent post on `linuxquestions.org` outlining the different methods to update the system to multilib. I've found that the first one listed (the official one) from the Slackware Wiki works fine. Just following the 'Quick n' dirty' instructions section works fine. Only read the detailed installation if you want to understand what is going on.

I prefer to use the third method listed: `slackpkg+`. Don't use the script in the compat directory though: it doesn't set multilib as priority for some reason, which caused all sorts of problems for me. Follow the instructions `phenixia2003` gives in his post instead. 

## Upgrade packages

The [systemupgade](http://docs.slackware.com/howtos:slackware_admin:systemupgrade) wiki article covers all the steps. 

One thing I found annoying is that slackpkg `install-new` asks to upgrade packages I don't want on my system (e.g., kde, xfce). To stop getting prompts to install such packages, edit `/etc/slackpkg/blacklist`:

```
kde/*
kdei/*
xfce/*
```

If using `UEFI` and `elilo`, it is safe to upgrade kernel without blacklisting, because the old kernel will still boot until you explicity activate the new one. See **"Using a generic kernel on UEFI systems"** heading above for how to do that.

## Support for nVidia Optimus

Someone made a one-liner to install it. See the [docs](http://docs.slackware.com/howtos:hardware:nvidia_optimus).
