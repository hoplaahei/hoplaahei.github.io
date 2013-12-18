---
layout: post
published: false
---

1. Backup your existing .emacs and .emacs.d folders
2. Install emacs-prelude (a preconfigured Emacs for beginners):
	curl -L http://git.io/epre | sh
3. Install quicklisp to your home directory:
	curl -O http://beta.quicklisp.org/quicklisp.lisp
    sbcl --load quicklisp.lisp
    (quicklisp-quickstart:install)
    (ql:quickload "quicklisp-slime-helper")

Now restart emacs and load slime with M-x slime (Alt-x slime).