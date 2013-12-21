---
layout: post
published: true
---

Emacs from the terminal does not have access to X11 libraries, so e.g., copying & pasting some code from Emacs to your web browser will not work. Using third party tools with Emacs can fix this.

Install xclip in your OS's package manager, then in emacs run:

```
M-x package-list-packages
```

Use C-s to search for xclip and install it.

Now add to ~/.emacs:

```
(xclip-mode 1)
```
Remember to restart Emacs.