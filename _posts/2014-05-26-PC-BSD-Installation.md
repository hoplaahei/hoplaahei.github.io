---
layout: post
published: false
---

PC-BSD has two types of release: 'RELEASE' and 'STABLE'

## RELEASE

This is the only version directly linked from the front page of the main site, and is actually stabler than 'STABLE' release. Use this if you do not want your system to change dramatically over time. It receives security updates and bug fixes, but software and drivers do not get changed to the latest versions by an update manager. If you later decide that you prefer having newer applications and features, then you must configure the system to switch to 'STABLE' (a rolling-release) or wait for the next 'RELEASE' candidate. 

## STABLE

Confusingly, this ISO is actually less stable than the 'RELEASE' version because it regularly updates software and drivers to the latest versions. Use this if you want to test the latest features of software, or get an updated driver that fixes your hardware. Do not use 'STABLE' release if you hate software that regularly changes how you work and interact with it. 

## Getting the ISOs

PC-BSD uses a Content Distribution Network (CDN) to get the ISOs from multiple locations without needing to select a mirror. Unfortunately, most browsers do not download from multiple locations by default, which results in a slower download. To fix this, install an extension that supports multiple download sources. Here is how you can do this in Firefox:

1. Install DownloadThemAll! extension
2. Navigate to the [CDN](iso.cdn.pcbsd.org)
3. Choose the latest stable release
4. Choose your architecture (e.g., amd64 for 64-bit)
5. Open the .sha26 file (has the same name as the .iso)
6. Copy the long hash string
7. Right-click on the link for the hybrid .iso and choose 'Save Link with DownloadThemAll!'
8. Paste the hash key and set the type to sha256
9. Download the .iso