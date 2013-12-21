---
layout: post
published: true
---

Here is simple method to open a file in emacs as root from a terminal. Add this alias to your shell config (e.g., ~/.zshrc or ~/.bashrc):

```
alias E=emacsclient -t -e '(find-file "/sudo::/etc/passwd")'
```