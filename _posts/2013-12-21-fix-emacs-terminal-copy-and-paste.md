---
layout: post
published: false
---

Emacs from the terminal does not have access to X11 libraries so e.g., copying some code from Emacs to your X web browser will not work. Using third party tools with Emacs can fix this. 

Install xclip in your OS's package manager, then in emacs run:

```
M-x package-list-packages
```

Use C-s to search for xclip and install it.

Finally add to .emacs:

```
(xclip-mode 1)
```