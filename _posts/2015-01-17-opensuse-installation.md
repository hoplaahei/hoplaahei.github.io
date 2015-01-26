---
layout: post
published: false
title: openSUSE installation
---

Here are some notes on my openSUSE installation.

## Take a snapshot

It is nice to be able to rollback to a fresh install if we make mistakes, so create a backup:

```
sudo su
snapper create -d "Fresh"
```

## Install essential packages

For me, the most essential package when installing a new system is a clipboard manager to manage copy/paste history (we will do a lot of it):

```
zypper in parcellite
parcellite &
```

Enable some repositories for more choice:

```
zypper ar -f http://packman.inode.at/suse/openSUSE_13.2/ packman
zypper ar -f http://download.opensuse.org/repositories/X11:/Utilities/openSUSE_13.2/X11:Utilities.repo
```

```
zypper install firefox flash-plugin git make cc rox-filer dmenu mpv rxvt-unicode urxvt-perls emacs-x11 feh mate-power-manager sawfish gtk-chtheme gpick yast2-sound trayer xosd xosd-devel xfontsel
```

## Configure fonts

If fonts are installed to non-standard folders or sub-directories, make the X server aware of these dirs:

```
xset +fp /font/dir
```

Force the X server to rescan and detect the new dirs:

```
xset fp rehash
```

Older applications without fontconfig may not see new fonts. The font directories must be fontified and the cache updated:

```
xset +fp /font/dir
cd /font/dir
mkfontscale
mkfontdir
xset fp rehash
```

List installed fonts with `fc-list`.

Some applications without `XFT` support need to be configured with an `X Logical Font Description (XLFD)` string. `xfontsel` makes it easy to generate these strings graphically and offers a preview of what the font will look like.

An example of an XLFD:

```
-*-fixed-*-r-*-*-14-*-*-*-*-*-*-*
```

Or if the application supports `XFT`:

```
fixed:size=14
```

## Get sound working

Install `yast2-sound` module and then:

`Yast` -> `Sound` -> `Edit`

Allow it to autoconfigure. 

## Playback all popular formats in Web Browser

Use this [one-click](http://opensuse-community.org/) install. This is needed to get e.g., h264 playback working in Firefox.

The following config change is also needed for Firefox:

- enter `about:config` in the URL bar
- search for `media.gstreamer.enabled`
- set it to `true`

## Enable Optimus Graphics Switching

Follow [this](https://en.opensuse.org/SDB:NVIDIA_Bumblebee) guide.

Don't forget to follow the step to blacklist `nouveau`. I ignored it and encountered problems such as module load errors on reboot, and 'Permission denied' messages when running `optirun`.

## Add user to groups

```
usermod -G video,audio,wheel,bumblebee joe
```

## Make some adjustments to a few things that irk me

1.) Set bootloader wait from 8 seconds to 1:

`Yast` -> `Bootloader` -> `Bootloader Options`.

2.) `sudo` without a password:

`Yast` -> `Sudo`

Edit `ALL` and check the box for `NOPASSWD`, so that it says 'Yes'. Also make sure your user is in the `wheel` group.

3.) Run graphical programs as root:

The following advice from openSUSE 'Login as root' document is meant to work, but doesn't for me:

```
sudo visudo
```

Prepend `DISPLAY` and `XAUTHORITY` to the variables in env_keep:

```
Defaults        env_keep = "DISPLAY XAUTHORITY LANG LC_ADDRESS ...
```

What did work for me is adding `xhost` command to `~/.bashrc`:

```
echo 'xhost + && clear' >> $HOME/.bashrc
```

4.) Get a better icon theme by editing `~/.gtkrc.mine`:

```
gtk-icon-theme-name="Bluecurve"
```
You need to download and extract [Bluecurve](http://gnome-look.org/content/show.php/Bluecurve+GNOME%2BMATE+Theme?content=148927) icons into `~/.icons` directory.

5.) Fix Flash in Palemoon

```
mkdir -p /usr/lib/mozilla
sudo ln -s /usr/lib64/browser-plugins/ /usr/lib/mozilla/plugins
```

6.) Get dependencies for wallpaper to .Xresources colour generating [script](http://charlesleifer.com/blog/using-python-to-generate-awesome-linux-desktop-themes/)

```
zypper install python-devel libjpeg8-devel python-pip
pip install Pillow
python colorscheme.py
```

7.) Remove Grub "sparse file" error


Comment these lines in `/etc/grub.d/00_header`:

```
cat << EOF
if [ -s \$prefix/grubenv ]; then
  load_env
fi
EOF
```

## Make Firefox less shit

`Treestyle Tabs` extension is nice, but I recommend setting the skin to 'Mixed' or 'Flat', otherwise you can't tell which tab is highlighted under certain GTK themes. It is also worthwhile to right click on the sidebar and choose 'Fix position and width of tab bar', or you will end up dragging it by accident. I also recommend going to the 'Tree' section of its configuration and unchecking 'When a new tree appears...' and 'When a new tab gets focus...', or you just end up loosing track of where your tabs got hidden.

Under `Advanced` of Firefox `Preferences`, I check 'Use autoscrolling' as this prevents me accidentally middle clicking on a page and activating the stupid clipboard URL load. 

## Why you should use a display manager

If you need autologin use something like `slim`, but if not then the default `xdm` that openSUSE uses on X only installations is good enough.

Example .xsession for `xdm` display manager:

```
#!/bin/bash
mate-power-manager &
xcompmgr &
feh --bg-scale ~/.wallpaper/current &
trayer --SetDockType false --transparent true --expand true --align right --alpha 255 &
parcellite &
sleep 1 &&
exec sawfish
```

And remember to:

```
chmod +x ~/.xsession
```

I do not suggest bypassing the displayer-manager service (using `startx` or `xinit` directly), even if you do autologin. The display manager is not just for logging users in; it does other things such as interface with `policykit` to enable `ACLs`, which many modern apps use to configure permissions. If you choose to bypass `policykit` and/or `ACLs`, then you may get unexpected behaviour, or some apps not working at all. 

But if you insist, here is the fiddly way to bypass using a display manager and still autologin to X...

## X autologin without display manager

Disable the current display manager in `/etc/sysconfig/displaymanager`:

```
DISPLAYMANAGER="console"
```

Now create a systemd drop-in file:

```
mkdir -p /etc/systemd/system/getty@tty1.service.d/
```

Edit `/etc/systemd/system/getty@tty1.service.d/autologin.conf`:

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin yourusername --noclear %I 38400 linux
Type=simple
```

Edit `~/.bash_profile`:

```
# Following automatically calls "startx" when you login:
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx
```

Uncomment this line in `/etc/permissions.local` to set suid bit:

```
#/usr/bin/Xorg                 root:root       4711
```

Run:

```
SuSEconfig --module permissions
```

And make sure your user is in any important groups such as audio and video (or things like Flash will do strange things).

## Create custom openSUSE packages

```
zypper in osc
```

Follow [this](https://en.opensuse.org/openSUSE:Build_Service_Tutorial) tutorial. The tutorial leaves out some cruical steps though. You must login to the build service via the web first and browse to the default home project. You will then get an option to generate it because it doesn't exist yet. **I haven't bothered looking up the equivalent cli commands to do this without logging in via the web.**

Also, it fails to note that the example code for adding a repository must be nested within the <project name=...> `XML` tags.

For packaging python modules use [this](https://en.opensuse.org/openSUSE:Packaging_Python) guide, but make sure you've followed the [Build Service](https://en.opensuse.org/openSUSE:Build_Service_Tutorial) tutorial first to setup repositories and projects correctly.

## Notes

When uninstalling software, remember to use: 

```
zypper remove --clean-deps
```