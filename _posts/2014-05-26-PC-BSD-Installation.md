---
layout: post
published: true
---

PC-BSD has two types of release. The 'RELEASE' is the default, downloadable from the main page. This is the stabler version, and only receives security updates and bug fixes between each major release. It is not, however, a rolling release, meaning you will have to instruct PC-BSD to upgrade to the next release.

Confusingly, there is also a 'STABLE' version, which is actually less stable than the 'RELEASE' version because it regularly updates software and drivers to the latest versions. Use this if you want to test the latest features of software, or get an updated driver that fixes your hardware. Do not use 'STABLE' release if you hate software that changes how you are used to working with it. 

PC-BSD uses a Content Distribution Network (CDN) to get the ISOs from multiple locations without needing to select a mirror. Unfortunately, Firefox does not download from multiple locations by default, resulting in a slower download. To fix this, install the DownloadThemAll! extension. Now navigate to the [CDN](iso.cdn.pcbsd.org) and choose the latest stable release and your architecture (e.g., amd64 for 64-bit). You will want the DVD-USB hybrid .iso. First look for the corresponding .sha26 file and either download or open it. Copy the long string (the 'hash'). Now right-click on the link for the hybrid .iso and choose 'Save Link with DownloadThemAll!'. In the box that appears look at the entry for the hash and set it to sha256. Copy and paste the text from the corresponding .sha256 file to the text field and you are ready to download at a faster speed.