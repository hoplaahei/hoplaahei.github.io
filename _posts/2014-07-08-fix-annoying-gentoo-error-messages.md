---
layout: post
published: true
---

## Grub Errors

Do you receive these errors?

```
No suitable mode found
```

or

```
Missing ... unicode.pcf
```

If so, follow these steps:

```
USE=truetype emerge grub
```

The `truetype` flag of `grub` is needed to get the `grub2-mkfont` executable. If you want this executable for every update of grub then you will need to add it to your `/etc/portage/package.use` permanently by adding the line:

```
sys-boot/grub truetype
```

Now we need the `unifont` package:

```
emerge unifont
```

And we use the unifont tools to convert the .hex file to bdf format:

```
hex2bdf /usr/share/unifont/unifont.hex > unifont.bdf
```

And convert .bdf to .pf2:

```
grub2-mkfont -o unicode.pf2 unifont.bdf
```

You need to copy the .pf2 file to a `fonts` folder to the `grub prefix` folder. The prefix is usually `/boot/grub`, so try that if you're not sure:

```
mkdir /boot/grub/fonts
sudo mv unicode.pf2 /boot/grub/fonts
```

And create a `/boot/grub/custom.cfg` file containing:

```
insmod font

if loadfont ${prefix}/fonts/unicode.pf2
then
    insmod gfxterm
    set gfxmode=auto
    set gfxpayload=keep
    terminal_output gfxterm
fi
```

Creating a custom.cfg prevents the settings getting overwritten on a Grub update. There is no need to regenerate anything.

## Kernel Errors

Do you receive this error message?

```
drivers/rtc/hctosys.c: unable to open rtc device (rtc0)
```

Disable `CONFIG_RTC_HCTOSYS` in kernel using a graphical tool:

```
genkernel --splash --install --menuconfig all
```

Look for `Device Drivers` -> `Real Time System Clock` and disable it with spacebar. Or press `/` to search for `CONFIG_RTC_HCTOSYS` and it will tell you where that option is located in the tree. See [here](https://github.com/raspberrypi/linux/issues/163) for explanation of what the RTC is used for, and whether or not you need it. Most people don't.