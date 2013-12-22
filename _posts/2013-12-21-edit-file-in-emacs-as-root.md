---
layout: post
published: true
---

The simplest way to edit root files in emacs is to download sudo-save.el.

Advantages: 

- only needs requiring in the init file
- autodetects when a root file is opened both from within emacs or directly from the commandline

Disadvantages:

- doesn't prompt for password
- requires [NOPASSWD](http://www.andrehonsberg.com/article/linux-sudo-without-a-password-using-the-sudoers-file) option in /etc/sudoers (security risk)
- or requires recent use of sudo command before timeout (in cache)


I still prefer this method despite the security risks because I'm lazy and like the audodetection rather than having to manually specifying that I'm editing a root file.

** Installation

```
wget http://dryice.name/computer/emacs/packages/sudo-save.el
```
And make sure you put the above in the same folder as your emacs loadpath and tell emacs to require it. If you do not know what your load path is then you can add on your own in ~/.emacs with e.g.,:

```
(add-to-list 'load-path "~/.emacs.d/personal/modules")
```

And you can require it by adding this in ~/.emacs:

```
(require 'sudo-save)
```

If you do not want to use the above elisp code then you can open as root directly from the terminal by adding this to e.g., ~/.zshrc or ~/.bashrc:

```
alias E=emacsclient -t -e '(find-file "/sudo::/etc/passwd")'
```
I'm yet to find a piece of elisp code that can do the same autodetection of root files as sudo-save.el, but with the added security of password prompting. There is [sudo.el](http://www.emacswiki.org/emacs/SudoSave), but it breaks for me when I try to open root files with emacs from the commandline. I'd be interested to know if anyone has came up with a working solution. 