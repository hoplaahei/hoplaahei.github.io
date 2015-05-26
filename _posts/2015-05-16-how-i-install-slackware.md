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

This step is not at all essential to installing Slackware, but it is something I always do out of habit. The tl;dr of SSD memory cell clearing is that it could reset any slow down in SSD write speeds by returning it to factory defaults (this blanks the disk):

```
echo -n "mem" > /sys/power/state # suspend to RAM to unfreeze security
hdparm --user-master u --security-set-pass PasSWorD /dev/sdX
hdparm --user-master u --security-erase PasSWorD /dev/sdX
```

The `ArchWiki` has a good [article](https://wiki.archlinux.org/index.php/SSD_memory_cell_clearing) on this, so I'm not going to reiterate the steps in detail.

## Partitioning

The SlackDocs wiki has an [install](http://docs.slackware.com/slackware:install) guide for standard installation. Follow these steps first. Some modern computers force the use of UEFI boot with a GPT partitioning scheme, and if you require/want this then see the UEFI [README](http://slackware.mirrorcatalogs.com/slackware64-14.1/README_UEFI.TXT).

If using the DVD ISO on a USB pen, tell the installer the files are on a USB when it asks, and it will scan automatically. Any EFI and swap partitions get detected and formatted automatically.

For LVM partitioning, there are some additional steps needed before and after setup, so read `README_LVM.txt` (included on the USB and readable from the console) carefully. The guide is excellent. I recommend scrolling down to the "alternative method" that automatically generates the right commands to pass to `mkinitrd`. Be warned, on `UEFI` systems, the generated command will not copy the files over to the `EFI` partition, or set the right paths in `elilo.conf`. This is easily fixable (see the next heading).

## Using a generic kernel on UEFI systems

There are some additional steps to switch from a huge kernel to a generic one on UEFI systems (using `elilo`) that the `beginners guide` doesn't mention. Copy the `initrd` and `vmlinuz` from that kernel to `/boot/efi/EFI/Slackware`, e.g., 

```
cp /boot/vmlinuz-generic-3.10.17 /boot/efi/EFI/Slackware/
cp /boot/initrd.gz /boot/efi/EFI/Slackware/
```

And edit `/boot/efi/EFI/Slackware/elilo.conf` as opposed to `/etc/lilo.conf`. The syntax is mostly the same as in the beginners guide example, but the `/boot/` part of the paths need removing, e.g.,:

```
image = vmlinuz-generic-3.10.17
        initrd = initrd.gz
        root = /dev/sdb3
        label = 3.10.17
        read-only
        append="vga=normal ro"
```

Leave any old entries in there above the new one, so you can select which kernel to boot with the arrow keys from the menu on boot. Once sure it boots, add this line after `timeout=1` in the `elilo.conf`:

```
default=3.10.17
```

And replace the above example with the `label` of the boot entry you need to boot.

## Enable resume from hibernation

In `/etc/lilo.conf` (non-UEFI BIOS setup) or `/boot/efi/EFI/Slackware/elilo.conf` (UEFI setup) add this somewhere in the double quotes ("") of the `append=` line:

```
resume=/dev/sdX # where X is the swap partition
```

## Get correct keys on the keyboard

After logging in it is useful to start `X` with `startx` command to run the default window-manager selected in installation e.g., `windowmaker`, but the keys might not be right on the keyboard. For a quick fix, run from the terminal e.g.,

```
setxkbmap gb # for a British keyboard
```

This will only work for the current session, but we will set it permanently later.

## Fix incorrect time

 [This](http://docs.slackware.com/howtos:hardware:syncing_hardware_clock_and_system_local_time) SlackDoc explains how to keep the time of all dual-booted operating systems in sync.

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

I advise to read the comments of `/etc/profile.d/lang.sh` when setting a locale, and follow the recommendation not to enable UTF-8 unless absolutely necessary. I experienced strange behaviour with some applications and UTF-8, e.g., `xpdf` failing to load some PDFs. If, however, UTF-8 is essential to your system, then you can still get the problem apps to load by overriding the locale per-application with `C` locale like this:

```
LANG=C xpdf
```

## Learn to use Slackbuilds

[Install](http://www.sbopkg.org/downloads.php) `sbopkg` and sync with the remote repository by running:

```
sbopkg -r
```

Now follow this guide to [manage queue files easily](http://slackblogs.blogspot.co.uk/2014/01/managing-sbo-dependencies-easily.html).

## Convert to multilib (optional)

`wine`, `virtualbox`, `skype` and `google-earth` are some examples of programs that need a `multilib` system.

`slackpkg+` can enable `multilib` and keep it updated easily. Download the latest [slackpkg+](http://sourceforge.net/projects/slackpkgplus/files/) package. Install from the cli in the same directory as the downloaded file with:

```
installpkg slackpkg+-*.txz
```

At the time of writing, the README says it is possible to use the script `/usr/doc/slackpkg+-*/setupmultilib.sh`. This script didn't work properly for me, as it didn't uncomment the line to set the multilib repo as the priority, which had me scratching my head.

Instead, manually edit `/etc/slackpkg/slackpkgplus.conf`:

```
# Uncomment (and keep it first in the list)
PKGS_PRIORITY=( multilib )
...

# Add "multilib" (order doesn't matter here)
REPOPLUS=( multilib slackpkgplus restricted alienbob slacky )
...
# And also uncomment
MIRRORPLUS['multilib']=http://taper.alienbase.nl/mirrors/people/alien/multilib/14.1/
```

Now run:

```
slackpkg update gpg
slackpkg update
slackpkg upgrade gcc glibc
slackpkg upgrade-all # optional
slackpkg install multilib
```

The [official](http://alien.slackbook.org/dokuwiki/doku.php?id=slackware:multilib) method works fine too, and is good for those who prefer not to use a third-party package management tool such as `slackpkg+`. Just follow the **'Quick n' dirty'** instructions section of the linked wiki page. It is not necessary to also complete the 'Detailed' instructions further down the page, but it might help in understanding what is going on. 

[phenixia2003](https://www.linuxquestions.org/questions/slackware-14/slackware64-and-my-stupidity-4175484839/#post5066064) gives a good overview of all the available methods for installing multilib.

## Upgrade packages

The [systemupgrade](http://docs.slackware.com/howtos:slackware_admin:systemupgrade) wiki article covers all the steps. 

**Note:** if using `UEFI` and `elilo`, it is safe to upgrade kernel automatically without first blacklisting it, because the old kernel will still boot until you explicitly activate the new one by copying it over to the `EFI` partition. See **"Using a generic kernel on UEFI systems"** heading above for how to do that.

One thing I found annoying is that `slackpkg install-new` asks to upgrade from package sets I'd already deselected in setup (e.g., kde, xfce). To stop getting prompts to install such packages, edit `/etc/slackpkg/blacklist` with e.g.,:

```
kde/*
kdei/*
xfce/*
```

## Support for nVidia Optimus

Someone made a one-liner to install it that works for me. See the [docs](http://docs.slackware.com/howtos:hardware:nvidia_optimus).

## Backing up

Backing up for me is as simple as running `rsnapshot manual` (I prefer this to an automatic cron job).

[rsnapshot](http://rsnapshot.org/) has LVM support built-in to its config file. My `rsnapshot` config makes use of an LVM snapshot volume to take a consistent, atomic (point-in-time) snapshot of the system before running rsync. This allows me to backup the root filesystem live, without having to unmount it first, and gives the assurance that write processes are allowed to finish before the backup is taken. Here are the important parts of my `/etc/rsnapshot.conf`:

```
config_version  1.2

snapshot_root   /media/backup/rsnapshot/
retain  manual  30
no_create_root  1
cmd_rm          /usr/bin/rm
cmd_rsync       /usr/bin/rsync
cmd_logger      /usr/bin/logger
cmd_du          /usr/bin/du

interval        hourly  6
interval        daily   7
interval        weekly  4
#interval       monthly 3

verbose         4

loglevel        3

lockfile        /var/run/rsnapshot.pid

one_fs          1

exclude dev/
exclude media/
exclude mnt/
exclude proc/
exclude sys/
exclude tmp/
exclude run/
exclude home/Downloads
exclude home/.gvfs


# LOCALHOST
linux_lvm_cmd_lvcreate  /sbin/lvcreate
linux_lvm_cmd_lvremove  /sbin/lvremove
linux_lvm_snapshotsize  5G
linux_lvm_snapshotname  rsnapshot-tmp
linux_lvm_vgpath        /dev
linux_lvm_mountpath     /mnt/rsnapshot-tmp
linux_lvm_cmd_mount     /usr/local/bin/mount-wrapper
linux_lvm_cmd_umount    /bin/umount

backup  lvm://slack/root/       slack-root/
```

This config works without LVM, but without its snapshotting capability, I recommend unmounting `/` and backing up from another OS, otherwise running processes might only partially complete writing to files, resulting in a tarnished backup.

Some explanation of the options:

- `snapshot_root` :: This is the directory where the backup will happen. I mount an external drive here to keep the backup offsite and save space. 
- `no_create_root  1` :: Prevent rsnapshot automatically creating the backup directory if it doesn't exist. Useful to make sure the external drive containing the backup directory is mounted (otherwise the backup would just create the directory automatically on the source drive and backup to the wrong place.)
- `one_fs          1` :: Don't go across to other mounted filesystems (so prevent also backing up e.g., mounted USB pens and external disks)
- `retain  manual  30` :: Keep 30 manual backups before starting to replace them
- `verbose         4` :: Print out which files are currently copying, so I know the backup is still running ok
- `linux_lvm_cmd_mount` ::  In later versions of `rsnapshot` it is possible to pass mount options here, but not in the release version on Slackware 14.1, so I point this option to a mount-wrapper script. 

Contents of the wrapper script `/usr/local/bin/mount-wrapper:

```
#! /bin/bash
#
# This is a wrapper for mount to pass it the nouuid option.
# This is required to enable rsnapshot to mount an xfs lvm snapshot.

/bin/mount -o nouuid $1 $2
if [ "$?"-ne 0]; then echo "Error detected. This wrapper only works with XFS filesystems"; exit 1; fi
```
I need this workaround because `XFS` will not mount filesystems with the same `UUID`. 