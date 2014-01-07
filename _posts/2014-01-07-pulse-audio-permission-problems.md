---
layout: post
published: false
---

Pulse audio may get permission errors if `/run/user/` sub-directory it creates is not `chown`'d to the user. Running pulseaudio -D will reset the permissions if some other misbehavings apps has changed the permissions of the `/run/user/` sub-dirs. 
