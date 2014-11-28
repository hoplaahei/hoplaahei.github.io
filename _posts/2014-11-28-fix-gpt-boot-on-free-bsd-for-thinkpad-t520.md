---
layout: post
published: true
---

There is a long withstanding bug in the bioses for T520, T420 and W520 that prevents the booting of operating systems that use a GPT partition scheme with type `ee00` (protective MBR) as the first partition. [Many](http://forums.lenovo.com/t5/Linux-Discussion/Lenovo-Thinkpad-T520-doesn-t-boot-with-GPT-slices-on-FreeBSD-9/td-p/555317) users have this problem. To workaround the issue you must boot a FreeBSD live disc/image and follow [this](http://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html) guide to modify the partition table of the existing installation you need to fix.