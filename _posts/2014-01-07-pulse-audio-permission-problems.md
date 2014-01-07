---
layout: post
published: true
---

If a misbehaving app has messed with the ownership of `/run/user` sub-directories then your pulse-audio server may not start. Running `pulseaudio -D` will set the ownership back to user for pulse-audio and allow it to start.