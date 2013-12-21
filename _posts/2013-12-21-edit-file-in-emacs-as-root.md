---
layout: post
published: true
---

There are a few elisp snippets floating about the web to save files as sudo, but most only allow you to open files with sudo from within emacs find-file, and not straight from a terminal. sudo.el does both. Download it:

```
wget --no-check-certificate https://raw.github.com/alexander-yakushev/.emacs.d/master/sudo.el
```

And make sure you put the above in the same folder as your emacs loadpath. If you do not know what your load path is then you can add on your own in ~/.emacs with e.g.,:

```
(add-to-list 'load-path "~/.emacs.d/personal/modules")
```

You also need the following snippet in your ~/.emacs

```
;; Save files as root

(defun sudo-before-save-hook ()
  (set (make-local-variable 'sudo:file) (buffer-file-name))
  (when sudo:file
    (unless(file-writable-p sudo:file)
      (set (make-local-variable 'sudo:old-owner-uid) (nth 2 (file-attributes sudo:file)))
      (when (numberp sudo:old-owner-uid)
        (unless (= (user-uid) sudo:old-owner-uid)
            (when (y-or-n-p
                    (format "File %s is owned by %s, save it with sudo? "
                             (file-name-nondirectory sudo:file)
                              (user-login-name sudo:old-owner-uid)))
                  (sudo-chown-file (int-to-string (user-uid)) (sudo-quoting sudo:file))
                      (add-hook 'after-save-hook
                                      (lambda ()
                                        (sudo-chown-file (int-to-string sudo:old-owner-uid)
                                                          (sudo-quoting sudo:file))
                                        (if sudo-clear-password-always
                                                (sudo-kill-password-timeout)))
                                            nil   ;; not append
                                            t    ;; buffer local hook
                                                  )))))))


(add-hook 'before-save-hook 'sudo-before-save-hook)
```

If you do not want to use the above elisp code then you can open as root directly from the terminal by adding this to e.g., ~/.zshrc or ~/.bashrc:

```
alias E=emacsclient -t -e '(find-file "/sudo::/etc/passwd")'
```