---
layout: post
published: true
---

- Move your existing .emacs file and .emacs.d folder
- Install sbcl
- Install emacs-prelude (a preconfigured Emacs for beginners):

```
 curl -L http://git.io/epre | sh
```

- Install quicklisp to your home directory:

```
curl -O http://beta.quicklisp.org/quicklisp.lisp
sbcl --load quicklisp.lisp
```
- From within the new sbcl buffer, paste these lines one-at-a-time, pressing enter after each:

```
(quicklisp-quickstart:install)
(ql:add-to-init-file)
(ql:quickload "quicklisp-slime-helper")
```
- Ignore the recommendations from quicklisp as prelude should have everything setup
- Restart emacs and load slime with M-x slime (alt-x slime)