---
layout: post
published: false
title: Setting up a Fresh Manjaro Install
---

# Before anything (crucial)

```
sudo pacman -Syu
```

# sudo without entering password

```
sudo visudo
```

```
Defaults:USER_NAME      !authenticate
```

Logout and back in.

# Nice zsh config

```
yaourt -S grml-zsh-config-git
```

# Faster download speeds

```
rankmirrors -g
```

# Fix resume from hibernation

Add `resume` hook to `/etc/mkinitcpio.conf`, e.g.,:

```
HOOKS="base udev autodetect modconf block resume filesystems keyboard keymap fsck"
```

Find out your kernel version with `uname -r`, e.g., mine is `3.18.11-1-MANJARO`.

Run `mkinitcpio` for that kernel version:

```
sudo mkinitcpio -p linux318
```

Find out the `PARTUUID` by looking at the swap partition entry in your `/etc/fstab`.

Edit `/etc/default/grub` to use this `PARTUUID`, e.g.,:

```
GRUB_CMDLINE_LINUX_DEFAULT="resume=PARTUUID=6da102e7-37c8-44dc-b48d-90ec09ebf9a3"
```

Regenerate `grub` with:

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
# Daemonize URxvt

The example on the Arch wiki using sockets doesn't work for me. Creating these files does:

`~/.config/systemd/user/urxvtd.service`

```
[Unit]
Description=RXVT-Unicode Daemon
Requires=urxvtd.socket

[Service]
ExecStart=/usr/bin/urxvtd -q -o
Environment=RXVT_SOCKET=%h/.urxvt/urxvtd-%H

[Install]
WantedBy=default.target
```

`~/.config/systemd/user/urxvtd.socket`

```
[Unit]
Description=urxvt daemon (socket activation)
Documentation=man:urxvtd(1) man:urxvt(1)

[Socket]
ListenStream=%h/.urxvt/urxvtd-%H
SocketMode=0700
Accept=yes

[Install]
WantedBy=sockets.target
```

And enable the service:

```
systemctl --user enable urxvtd
systemctl --user start urxvtd
```

# Wifi not enabled on startup

Upgrade to at least kernel 3.18x

```
sudo mhwd-kernel -i linux318
```

# Manually add UEFI boot entry if you get errors during install

```
sudo efibootmgr -c -p 2 -d /dev/sda -L Manjaro -l "\EFI\manjaro_grub\grubx64.efi"
```

# Install virtualbox so its modules update automatically on kernel upgrade

Get the `linux-headers` for your kernel e.g.,

```
uname -r
```

Which might say we have `3.16.7.8-1-MANJARO`, so:

```
pacman -S linux316-headers
```

When installing `virtualbox` with `pacman -S virtualbox` choose 8) to select the dkms repo.

Now run this command:

```
dkms install vboxhost/$(pacman -Q virtualbox|awk '{print $2}'|sed 's/\-.\+//') -k $(uname -rm|sed 's/\ /\//')
```
Enable `dkms` service so you don't need to type the above command each time you upgrade the kernel:

```
sudo systemctl enable dkms.service
```

Enable `vboxdrv` without reboot:

```
sudo modprobe vboxdrv
```

Install the extension pack:

```
yaourt -S virtualbox-ext-oracle
```

Add yourself to the user group:

```
sudo gpasswd -a joe vboxusers
```

If `virtualbox` doesn't start try installing qt4:

```
sudo pacman -S qt4
```

# Fixing misconfigured graphics

See what drivers are available for your card:

```
mhwd
```

See what drivers are already installed:

```
mhwd -li
```

If the wrong drivers are installed (e.g., one time the installer selected nouveau for my card, but it didn't work) uninstall with:

```
sudo mhwd -r pci name-of-listed-config
```

Install the right one (e.g., `nvidia-bumblebee`):

```
sudo mhwd -i pci name-of-correct-config
```

The `mhwd` script didn't work first time for me because of conflicts with `xorg-server`, so I temporarily removed it:

```
sudo pacman -R xorg-server
```

And it was pulled back in by the `mhwd` script anyway, so no harm done (I think).

# Get rid of black Manjaro themed colours in Firefox

For some reason changing the `GTK` theme does not get rid of Manjaro dark themed colours in Firefox (the URL and URL search have a black background). I removed my whole `.mozilla` directory and it fixed it, but I'd be interested to know a more precise way to fix it, one which doesn't involve losing any Firefox settings not backed up by `Mozilla Sync`.

# Problems with AUR builds

The minimal Manjaro install doesn't include `AUR` support for some reason. I tried installing `base-devel`, but that conflicted with the multi-lib support, so I ended up having to install most of the base-devel packages manually (e.g., autoconf aclocal) to get things to compile . It was a bit of a pain to search for the multilib equivalents of some of these packages when I found conflicts. 

# Set Thinkpad battery thresholds

Get the `linux-headers` for your kernel e.g.,

```
uname -r
```

Which might say we have `3.16.7.8-1-MANJARO`, so:

```
pacman -S linux316-headers
```

Install acpi_call_dkms from the `AUR`:

```
yaourt -S acpi_call-dkms
```

Follow the instructions it prints out e.g.,:

```
sudo dkms install acpi_call/1.1.0
sudo systemctl enable dkms.service
```

This will build `acpi_call` modules automatically when a new kernel is installed.

If you do not want to reboot run:

```
sudo modprobe acpi_call
```

Now install tpacpi-bat:

```
yaourt -S tpacpi-bat
```

And set the thresholds:

```
sudo /usr/bin/perl /usr/bin/tpacpi-bat -v -s SP 0 80
sudo /usr/bin/perl /usr/bin/tpacpi-bat -v -s ST 0 40
```

As the battery scripts require elevated priveleges, create a `systemd` system-wide service to run this at startup (in case the thresholds get reset by taking the battery out the laptop):

```
nano -w /etc/systemd/system/set-battery.service
```

Paste this over to the service file:

```
[Unit]
Description=Set battery capacity

[Service]
Type=oneshot
ExecStart=/usr/bin/perl /usr/bin/tpacpi-bat -v -s SP 0 80
ExecStart=/usr/bin/perl /usr/bin/tpacpi-bat -v -s ST 0 40

[Install]
WantedBy=multi-user.target
```

And enable the service:

```
sudo systemctl enable set-battery.service
```

# Setup a tor proxy without DNS leaks

Install and enable both `tor` and `polipo`:

```
sudo pacman -Syy
sudo pacman -S tor polipo
sudo systemctl enable tor polipo
```

Copy the example polipo config over for editing:

```
cp /etc/polipo/config.sample /etc/polipo/config
```

Now edit it to make polipo use tor:

```
socksParentProxy = localhost:9050
socksProxyType = socks5
```
And start the tor and polipo services:

```
sudo systemctl start polipo tor
```