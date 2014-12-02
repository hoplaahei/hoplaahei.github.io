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
dbus_enable="YES" # you'll probably want this too
```

I get Xorg running without `HAL` instead, as I don't use anything hefty like `GNOME`:

```
Xorg -config /usr/local/etc/X11/xorg.conf
```

And add this line to the `ServerLayout` section of the newly generated `/usr/local/etc/X11/xorg.conf` to disable `HAL`:

```
Option       "AutoAddDevices" "Off"
```

Now install a window manager:

```
pkg install sawfish
```
Get `sawfish` to run when `X` starts with `ee ~/.xinitrc`:

```
exec sawfish
```
From now on I prefer to edit with something more robut than `ee`. I will use `ports` to install `emacs` with `lucid` toolkit rather than `GTK` (because for years emacsclient has been crashing with 100% CPU usage if you close Xorg while a graphical GTK frame is running).

```
cd /usr/ports/emacs
make clean
```

If that directory isn't found for whatever reason, install the ports `tree` with:

Might as well get a browser window open for reference as well:

```
pkg install firefox
```


