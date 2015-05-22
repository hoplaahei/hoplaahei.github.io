---
layout: post
published: true
title: Why I use Slackware
---

Slackware is great for people like me who are OCD over keeping full control of a system. Although it offers automated tools and scripts for repetitive tasks, it also mandates more user interaction and understanding of the different Linux components than most other distros. To me this is a good thing, because automated tools don't always do what you expect, and in the long term often lead to undesirable changes and untraceable breakage.

The thing that puts many off using Slackware is the lack of officially supported dependency resolution for packages. In practice, however, dependency conflicts rarely happen, due to a large amount of software available as Slackbuilds. Slackbuilds require only the base system or other Slackbuilds as dependencies, so nothing tends to conflict.

`sbopkg` makes installing Slackbuilds easy. It will detect modifications you make to the Slackbuilds and offer a choice between using the original or edited version. This prompt helps to remind the user of any changes made. Furthermore, any options set only apply to that Slackbuild, and will not change the defaults for other Slackbuilds, which is reassuring to know for a control freak such as myself.

The installer scripts of some other distros are complex and overuse syntactic sugar; this sugar helps package maintainers who do a lot of packaging, but adds unecessary layers of abstraction and complexity for the average user. Conversely, Slackbuilds use simple, standard, but well documented shell script. Most scripts make smart choices by including shell variables in the head to make it easy for the user to discover and set the interesting features of software.

There is no need to mess about with compile time options and dependencies if you don't want to either. If you don't care about fine-tuning which dependencies get used, `sbopkg` can use [queue](http://www.sbopkg.org/queues.php) files to find the right Slackbuilds and install them as dependencies in the correct order. I've never had a problem installing a Slackbuild in this manner. `sqg` tool is used to keep an up-to-date list of queue files for all Slackbuilds. 

Also, the full base install of Slackware includes many of the useful command line tools and development tools one might gradually end up installing anyway. Slackers encourage the use of a 'Full' install, as it includes many of the development files needed for building and installing software. Only on rare occasions have I needed to compile a package manually, and even then there are tools such as `src2pkg` to make Slack packages from any source.

When updating a stable Slackware installation, you still get security patches and newer versions of programs, but there are never any drastic changes between releases. The base system mostly remains the same (especially in the stable branch), and if you stick to using Slackbuilds or well vetted third-party packages (such as those from the Alien BOB repos), breakage is unlikely.

So if you want a system that you can tinker with as you please, but still actually rely on for daily work and play, please take a look at Slackware.
