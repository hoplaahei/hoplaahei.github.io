---
layout: post
published: false
title: Setting up a Slackware installation
---

# Basic configuration (users, locale, etc)

Follow the [beginners guide](http://docs.slackware.com/slackware:beginners_guide).

# Run NetworkManager and start on next reboot

```
chmod +x /etc/rc.d/rc.networkmanager
/etc/rc.d/rc.networkmanager start
```

# Configure sbotool

Setup `sbopkg` to [automatically](http://slackblogs.blogspot.ca/2014/01/managing-sbo-dependencies-easily.html) offer to install all the dependencies of a `slackbuild` for you. This is NOT the same as dependency management, but it will save you the bother of installing each dependency individually, and it will also install the dependencies in the right order.

# Blacklist unwanted updates

I don't want updates for XFCE or KDE (don't have them installed) so:

```
slackpkg blacklist kde kdei xfce
```

# Powersave by using nVidia Optimus

Follow the simple `optimus` [guide](http://docs.slackware.com/howtos:hardware:nvidia_optimus).

# Make bootloader aware of hibernation (swap) partition

- for `UEFI` systems edit `/boot/efi/EFI/Slackware/elilo.conf`
- for `BIOS` systems edit `/etc/lilo.conf`

```
append="root=/dev/sdb3 vga=normal resume=/dev/sdb2 ro"
```
Here we have added the `resume` stipulation, pointing to our `swap` partition.

On a `BIOS` system update your changes with:

```
lilo
```

UEFI systems do not need to update the bootloader.

# Upgrade the system

See 'Full system upgrade' in [slackpkg](http://docs.slackware.com/slackware:slackpkg) guide.

# Fix time not syncing

Change the servers to something nearer in `/etc/ntp.conf`:

```
	   server 0.uk.pool.ntp.org
	   server 1.uk.pool.ntp.org
	   server 2.uk.pool.ntp.org
	   server 3.uk.pool.ntp.org
```

# Fix xterm not copy/pasting from clipboard

Edit `$HOME/.Xdefaults`:

```
XTerm*selectToClipboard: true
xterm.VT100.Translations:    #override \n\
        Shift <KeyPress> Insert:        insert-selection(CLIPBOARD) \n\
        <Btn2Up>:                       insert-selection(SELECT, CUT_BUFFER0) \n\
        ~Shift<BtnUp>:                  select-end(PRIMARY, CUT_BUFFER0) \n\
        Shift<BtnUp>:                   select-end(CLIPBOARD, CUT_BUFFER1)
```