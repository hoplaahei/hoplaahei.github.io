---
layout: post
published: true
title: Advantages of manual dependency management in Slackware
---




Slackware is great for people like me who are OCD over keeping full control of a system. Although it offers automated tools and scripts for installing software and configuring the system, it encourages users to study the configuration options of the components, to better understand how to maintain a Linux system. 

Arguably the most distinctive characteristic of Slackware is the lack of officially supported dependency resolution for packages. This is a good thing, as automated dependency resolution eventually leads to undesirable changes and untraceable breakage. The Slackware way of leaving it up to the user to track dependencies is not as tedious as it might first seem, due to the growing amount of well maintained software available through Slackbuilds. Slackbuilds only require the base system or other Slackbuilds as dependencies, so nothing tends to conflict.

`sbopkg` makes installing Slackbuilds easy. It detects user modifications to the script, and offers a choice between using the original or edited version. This prompt helps to remind the user of any changes made. Furthermore, any options set only apply to that Slackbuild, and will not change the defaults for other Slackbuilds -- this is reassuring to know for a control freak such as myself.

The package build scripts of some other distros are complex, and overuse syntactic sugar; this sugar helps package maintainers who do a lot of bulk packaging, but adds unecessary layers of abstraction and complexity for the average user. Conversely, Slackbuilds use simple, standard, but well documented shell script. Variables listed at the start of Slackbuilds make it easy for the user to discover and set the interesting features of software.

If you don't care about fine-tuning which dependencies get used, `sbopkg` can use [queue](http://www.sbopkg.org/queues.php) files to find the right Slackbuilds and install them as dependencies in the correct order. `sqg` tool is used to keep an up-to-date list of queue files for all Slackbuilds. The combination of queue files and Slackbuilds works surprisingly well. 

Also, the full base install of Slackware includes many of the useful command-line tools and development tools that end up getting installed gradually on typical systems anyway. The 'Full' Slackware install is recommended, as it includes many of the development files needed for building and installing software. On the rare occasions when no Slackbuild is available, `src2pkg` helps make Slack packages from any source.

When updating a Slackware installation, security patches and newer versions of programs are installed, but there are never any drastic changes between releases. The base system mostly remains the same (especially in the stable branch), and by sticking to Slackbuilds or well vetted third-party packages (such as those from the Alien BOB repos), breakage is unlikely.

For a stable system that you can rely on for daily work and play, take a look at Slackware.
