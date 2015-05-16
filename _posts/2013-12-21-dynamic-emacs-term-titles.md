---
layout: post
published: false
title: Untitled
---


Use this method to change the terminal title to the running emacs buffer. I've prefixed the titles with '[Emacs]', which is useful if you want a unique set of symbols to match with window-matching rules in your window manager e.g., for cycling between open emacs terminal windows. 

Add this to ~/.emacs init file:

```
  (defun xterm-title-update ()
    (interactive)
    (send-string-to-terminal (concat "\033]1; " "[emacs] " (buffer-name) "\007"))
    (if buffer-file-name
        (send-string-to-terminal (concat "\033]2; " "[emacs] " (buffer-file-name) "\007"))
        (send-string-to-terminal (concat "\033]2; " "[emacs] " (buffer-name) "\007"))))

(add-hook 'post-command-hook 'xterm-title-update)
(add-hook 'window-configuration-change-hook 'xterm-title-update)
```

If using tmux you will also need (in ~/.tmux.conf):

```
set -g set-titles on
set -g set-titles-string "#T"
```
