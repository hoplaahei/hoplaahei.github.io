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


I still prefer this method despite the security risks because I'm lazy and like the audodetection, rather than manually specifying that I'm editing a root file.

## Installation

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
[sudo.el](http://www.emacswiki.org/emacs/SudoSave) is meant to offer the same as sudo-save, but with password prompting. Unfortunately, it does not switch read-only mode off when root files are opened from a terminal in emacsclient. And WARNING: if you try to set read-only flag off yourself and save the file it will work, but the file remains chowned as normal user, rather than root. 