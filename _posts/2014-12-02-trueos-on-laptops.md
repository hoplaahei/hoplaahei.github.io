---
layout: post
published: false
title: TrueOS Laptop Install
---

TrueOS is almost vanilla FreeBSD with a few tweaks, and cli versions of most PC-BSD tools. It is designed for server installations, but I like having its tools (such as `Life Preserver`) available on my laptops too. It also sets up a ZFS-On-Root quickly. 

First thing to do when booted into TrueOS is get the wifi up. In `/etc/rc.conf` ammend:

```
ifconfig_wlan0="DHCP"
```

Change it to:

```
ifconfig_wlan0="WPA DHCP"
```

This simple change will invoke `wpa_supplicant` on boot. Now edit `/etc/wpa_supplicant.conf` from the cli with:

```
wpa_passphrase yourAP yourPass > /etc/wpa_supplicant.conf
```
Restarting the network service after this change is finicky (at least on my Intel Centrino Ultimate-N 6300), so I found a reboot was needed to get the wifi adapter up and connected to the AP.

Having `moused` up and running is also useful for copy and pasting from configs while setting up. Enable it in `/etc/rc.conf`, and while we're at it, enable some basic laptop powersaving too:

```
moused_enable="YES"
powerd_enable="YES"
powerd_flags="-a hiadaptive -b adaptive"
```
This will use `Turbo Boost` CPU technologies when plugged in (`-a`), but save some power on battery (`-b`).

 >  hiadaptive:
 >  Like adaptive mode, but tuned for systems where performance
 >  and interactivity are more important than power consumption.
 >  It increases frequency faster, reduces frequency less aggres-
 >  sively, and will maintain full frequency for longer.  May be
 >  abbreviated as hadp.

Before I start making more complex changes to the system, I use `lpreserver` to make a `cron` job that takes `ZFS` backup snapshots:

```
lpreserver cronsnap tank start auto
```
See `lpreserver help cronsnap` for explanation.

> * Snapshots will be created every 5 minutes and kept for an hour.
> * An hourly snapshot will be kept for a day.
> * A daily snapshot will be kept for a month.
> * A monthly snapshot will be kept for a year.
> * The life-preserver daemon will also keep track of the zpool disk space. If the capacity falls below 75% the oldest snapshot will be auto-pruned.

Pull in the graphical environment with:

```
pkg install xorg-minimal
```

Install some required drivers e.g.,:

```
pkg install xf86-video-intel xf86-input-synaptics
```

If you need `HAL` (I don't bother with it), then edit `/etc/rc.conf`:

```
hald_enable="YES"
```

I get Xorg running without `HAL` instead, as I don't use anything hefty like `GNOME`:

```
Xorg -config /usr/local/etc/X11/xorg.conf
```

And add this line to the `ServerLayout` section of the newly generated `/usr/local/etc/X11/xorg.conf` to disable `HAL`:

```
Option "AutoAddDevices" "Off"
```

Now install a window manager and a terminal to run some commands in the graphical environment:

```
pkg install sawfish rxvt-unicode
```
Get `sawfish` WM to run with a terminal spawned when `X` starts by editing `ee ~/.xinitrc`:

```
exec urxvtcd & # start urxvt daemon if it isn't already running
exec sawfish
```
From now on I prefer to edit with something more robut than `ee`. I will use `ports` to install `emacs` with `lucid` toolkit rather than `GTK` (because for years emacsclient has been crashing with 100% CPU usage if you close Xorg while emacs GTK widgets are running).

```
cd /usr/ports/emacs
make clean
```

If that directory isn't found for whatever reason, install the ports `tree` with:

Might as well get a browser window open for reference as well:

```
pkg install firefox
```
This `Firefox` package also requires the enabling of `DBUS` in `/etc/rc.conf`:

```
dbus_enable="YES"
```


