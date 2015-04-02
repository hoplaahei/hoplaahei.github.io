---
layout: post
published: false
title: Setting up a Fresh Manjaro Install
---

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