---
layout: post
published: true
---

DISCLAIMER: Although the following works fine on my system, I cannot account for the quirks in some shells, and therefore urge you to backup your home directory before trying the following. 

In this guide I assume you know howto:

- create a github account
- setup a public key
- add a github repository
- use shell commands

Choose the dotfiles or dotfolders (.****) you want backing up from your home directory and move them to a folder ~/.dotfiles.

We need to remove the dots from the dotfiles/folders so they are visible in github. Run this loop in the terminal:

```
cd ~/.dotfiles && for f in * ; do mv "$f" "${f/./}" ; done
```
Now we symlink the renamed dotfiles back to their original home directory location:

```
cd ~/.dotfiles && for f in *; do ln -s ~/.dotfiles/$f ~/.$f; done
```

If you mess up and create a load of useless symlinks in the home dir then you can remove them quickly with:

```
find /home/joe -maxdepth 1 -lname '*' -exec rm {} \;
```

Now to make a local git repository:

```
cd ~/.dotfiles
git init
git add *
git commit -m "First dotfiles commit."
```

We also sync it with our remote repository (which you must create) on github:

```
cd ~/.dotfiles
git remote add origin git@github.com:rodyaj/dotfiles.git
git push -u origin master
```