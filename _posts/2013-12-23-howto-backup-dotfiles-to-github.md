---
layout: post
published: false
---

DISCLAIMER: Although the following works fine on my system, I cannot account for the quirks in some shells, and therefore urge you to backup your home directory before trying the following. 

In this guide I assume you know how to:

- create a github account
- [setup](https://help.github.com/articles/generating-ssh-keys) security keys
- add a github repository
- use shell commands

Choose the dotfiles or dotfolders (.****) you want backing up from your home directory and move them to a folder ~/.dotfiles:

```bash
mv .yourfile ~/.dotfiles
mv .yourfolder ~/.dotfiles
```

To remove the '.' from the dotfiles/folders so they are visible in github, run this loop in the terminal:

```bash
cd ~/.dotfiles && for f in * ; do mv "$f" "${f/./}" ; done
```

Symlink the renamed dotfiles back to their original home directory location:

```bash
cd ~/.dotfiles && for f in *; do ln -s ~/.dotfiles/$f ~/.$f; done
```

If you mess up and create useless symlinks in the home dir, then you can remove them quickly with:

```bash
find /home/joe -maxdepth 1 -lname '*' -exec rm {} \;
```

Make a local git repository:

```bash
cd ~/.dotfiles
git init
git add *
git commit -m "First dotfiles commit."
```

Sync it with our remote repository (which you must create) on github:

```bash
cd ~/.dotfiles
git remote add origin git@github.com:rodyaj/dotfiles.git
git push -u origin master
```

If you receive an error message that files already exist on the remote repository (e.g., README.md) then you can usually fix this with:

```bash
git pull origin master
git add *
git commit -m "Merge local and remote"
git push -u origin master
```

To restore the dotfiles from github:

```bash
cd ~
git clone git@github.com:rodyaj/dotfiles.git .dotfiles
cd .dotfiles && for f in *; do ln -s ~/.dotfiles/$f ~/.$f; done
```