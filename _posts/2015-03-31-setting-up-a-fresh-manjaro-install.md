---
layout: post
published: false
title: Setting up a Fresh Manjaro Install
---

# Manually add UEFI boot entry if you get errors during install

```
sudo efibootmgr -c -p 2 -d /dev/sda -L Manjaro -l "\EFI\manjaro_grub\grubx64.efi"
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
mhwd -r pci name-of-listed-config
```

Install the right one (e.g., `nvidia-bumblebee`):

```
mhwd -i pci name-of-correct-config
```

The `mhwd` script didn't work first time for me because of conflicts with `xorg-server`, so I temporarily removed it:

```
pacman -R xorg-server
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
yaourt -S acpi_call_dkms
```

Follow the instructions it gives you e.g.,:

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

Create a `systemd` user-service to run this at startup (in case the thresholds get reset by taking the battery out the laptop):

```
nano ~/.config/systemd/user/set-battery.service
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