---
layout: post
published: true
---

Adding this elisp code to ~/.emacs will re-open a file as root that you access with find-file:

```
(defun th-rename-tramp-buffer ()
  (when (file-remote-p (buffer-file-name))
    (rename-buffer
     (format "%s:%s"
             (file-remote-p (buffer-file-name) 'method)
             (buffer-name)))))

(add-hook 'find-file-hook
          'th-rename-tramp-buffer)

(defadvice find-file (around th-find-file activate)
  "Open FILENAME using tramp's sudo method if it's read-only."
  (if (and (not (file-writable-p (ad-get-arg 0)))
           (y-or-n-p (concat "File "
                             (ad-get-arg 0)
                             " is read-only.  Open it as root? ")))
      (th-find-file-sudo (ad-get-arg 0))
    ad-do-it))

(defun th-find-file-sudo (file)
  "Opens FILE with root privileges."
  (interactive "F")
  (set-buffer (find-file (concat "/sudo::" file))))
  ```
  
Unfortunately the above does not seem to work when opening a file from a terminal. But you can make an alias in your shell (e.g., ~/.zshrc or ~/.bashrc) to open a file as root from the shell manually:

```
alias E=emacsclient -t -e '(find-file "/sudo::/etc/passwd")'
```

I'd be interested to find an elisp snippet that combines the two methods into one. I'll have to remember to make my own when I have time. 