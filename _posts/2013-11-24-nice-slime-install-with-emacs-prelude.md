---
layout: post
published: true
---

- Backup your existing .emacs and .emacs.d folders
- Install emacs-prelude (a preconfigured Emacs for beginners):

```
 curl -L http://git.io/epre | sh
```

- Install quicklisp to your home directory:

```
curl -O http://beta.quicklisp.org/quicklisp.lisp
sbcl --load quicklisp.lisp
(quicklisp-quickstart:install)
(ql:quickload "quicklisp-slime-helper")
```
- Restart emacs and load slime with M-x slime (Alt-x slime)