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

If you use a different locale than US you may find lots of applications throw complaints on the console because they do not correctly respect locale settings. Sadly, there is not yet any magic syntax to fix this, and each variable needs setting individually at the end of `default` section in `/etc/login.conf`.

```
default:\
        :passwd_format=md5:\
        :copyright=/etc/COPYRIGHT:\
...
        :umask=022:\
        :lang=en_GB.UTF-8:\
        :setenv=LC_ALL=en_GB.UTF-8:\
        :setenv=LC_COLLATE=en_GB.UTF-8:\
        :setenv=LC_CTYPE=en_GB.UTF-8:\
        :setenv=LC_MESSAGES=en_GB.UTF-8:\
        :setenv=LC_MONETARY=en_GB.UTF-8:\
        :setenv=LC_NUMERIC=en_GB.UTF-8:\
        :setenv=LC_TIME=en_GB.UTF-8:\
        :charset=UTF-8:\
```
Remember to run `cap_mkdb /etc/login.conf` and logout and back in from your user session.

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

If you care about the 'green' energy saving of your laptop, setting `-a` flag of `powerd_flags` to `adaptive` rather than `hidapative` should save power even with the adapter plugged in, but the likes of `Intel Turbo Boost` will get disabled.

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

Copy the generate config to `/etc/X11/`:

```
cp /usr/local/etc/X11/xorg.conf /etc/X11/xorg.conf
```

And add this line to the `ServerLayout` section of `/etc/X11/xorg.conf` to disable `HAL`:

```
Option "AutoAddDevices" "Off"
```

You can also set keymap options by editing the `InputClass`. In this example I've added two `Option` lines. The first uses `Great Britain` keyboard layout, and the second makes `CapsLock` key a second `Ctrl` key:

```
Section "InputClass"
        Identifier "Keyboard0"
        Driver "kbd"
        Option "XkbLayout" "gb"
        Option "XkbOptions" "ctrl:nocaps"
EndSection
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
A video player is nice to start watching something while `emacs` finishes compiling:

```
pkg install mpv
```
Getting flashplayer working on `TrueOS` is very easy:

```
pkg install pipelight
pipelight-plugin --create-mozilla-plugins
pipelight-plugin --enable flash
```

## Powersaving

In `/etc/rc.conf` set wifi into powersave mode (may take slightly longer to uplink to routers):

```
ifconfig_wlan0="-powersave WPA DHCP"
```

Some `/boot/loader.conf` tweaks:

```
hint.p4tcc.0.disabled=1
hint.acpi_throttle.0.disabled=1

drm.i915.enable_rc6=7 # this setting for intel devices saves quite a bit of energy

hw.snd.latency=7

hint.apic.0.clock=0
kern.hz=100 # lower kernel interrupt important for C3 state. Not sure about others states
hint.atrtc.0.clock=0

cpufreq_load="YES"

```
I've read in a forum post that the `p4tcc` line is no longer necessary, but I've not seen any official opinion on this.

Reload `grub`:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

Some `/etc/rc.conf` tweaks:

```
# Powersaving Tweaks
powerd_enable="YES"
powerd_flags="-a hiadaptive -b adaptive -n adaptive"
performance_cx_lowest="Cmax"
economy_cx_lowest="Cmax"
```
The `Cmax` value confused me at first, but the higher the value the `C-state`, the more effective the powersaving is. Many FreeBSD powersaving tutorials say to set `C3` state, but using the `Cmax` setting was optimal for my i7. It appears to achieve `C8` state, i.e., `sysctl dev.cpu | grep cx` shows:

```
dev.cpu.0.cx_supported: C1/1/1 C2/2/80 C3/3/109
dev.cpu.0.cx_lowest: C8
dev.cpu.0.cx_usage: 14.95% 3.94% 81.10% last 2942us
dev.cpu.1.cx_supported: C1/1/1 C2/2/80 C3/3/109
dev.cpu.1.cx_lowest: C8
dev.cpu.1.cx_usage: 12.60% 3.23% 84.15% last 1529us
dev.cpu.2.cx_supported: C1/1/1 C2/2/80 C3/3/109
dev.cpu.2.cx_lowest: C8
dev.cpu.2.cx_usage: 12.01% 3.59% 84.39% last 4195us
dev.cpu.3.cx_supported: C1/1/1 C2/2/80 C3/3/109
dev.cpu.3.cx_lowest: C8
dev.cpu.3.cx_usage: 13.99% 4.36% 81.64% last 3165us
```
This change was the single most noticeable improvement on my battery life (it shot up from 2 hours to 5 hours).

## Fixing Thinkpad buttons, Fn keys & suspend/resume glitches

Put in `/boot/loader.conf`:

```
acpi_ibm_load="YES"
```

Reload `grub`:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

Edit `/etc/sysctl.conf`:

```
hw.acpi.reset_video=0
hw.acpi.lid_switch_state=S3
hw.acpi.sleep_button_state=S3
hw.acpi.power_button_state=S5
hw.acpi.sleep_delay=3
hw.acpi.verbose=1
hw.syscons.sc_no_suspend_vtswitch=0
dev.acpi_ibm.0.events=1
```
If using the `intel` graphics driver then install `acpi_call` to dim the brightness:

```
pkg install acpi_call
kldload acpi_call
acpi_call -p '\VBRC' -i 8
```
Make the changes permanent by adding to `/boot/loader.conf`:

```
acpi_call_load="YES"
```

Remember to reload`grub`:

```
grub-mkconfig -o /boot/grub/grub.cfg
```
You can then add the brightness dim to a key with e.g., `xbindkeys`.

If you have nVidia `discrete` graphics enabled permanently in BIOS then edit `/etc/X11/xorg.conf` to get brightness controls working:

```
Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"
        Option     "RegistryDwords" "EnableBrightnessControl=1"
EndSection
```