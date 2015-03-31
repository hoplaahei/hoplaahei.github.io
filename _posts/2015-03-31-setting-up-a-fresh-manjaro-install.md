---
layout: post
published: false
title: Setting up a Fresh Manjaro Install
---

## Setup a tor proxy without DNS leaks

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