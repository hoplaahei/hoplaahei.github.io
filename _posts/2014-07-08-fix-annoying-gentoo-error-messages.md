---
layout: post
published: false
---

Do you receive these errors:

> No suitable mode found

or

> missing ... unicode.pcf

If so, follow these steps:

```
USE=truetype emerge grub
```

The `truetype` flag of `grub` is needed to get the `grub2-mkfont` executable. If you want this executable for every update of grub then you will need to add it to your `/etc/portage/package.use` permanently e.g.,

```
sys-boot/grub truetype
```

Now we need the unifont package:

```
emerge unifont
```

And we use the unifont tools to convert the hex to bdf format:

```
hex2bdf /usr/share/unifont/unifont.hex > unifont.bdf
```

And convert .bdf to .pf2:

```
grub2-mkfont -o unicode.pf2 unifont.bdf
```

Now copy the .pf2 file over:

```
mkdir /boot/grub/fonts
sudo mv unicode.pf2 /boot/grub/fonts
```

Now create a /boot/grub/custom.cfg file containing:

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
