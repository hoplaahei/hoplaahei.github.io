---
layout: post
published: false
title: Praise for Slackware and Slackbuilds
tags: 
  - "null"
---


Slackbuilds make it easy to maintain and update computer software without too
much problem solving. They tend to make every effort possible to rely on the
base Slackware install for dependencies (for that reason a full installation is
recommended). Where that is not possible, other Slackbuilds are pulled in, but
their quality ensures that this is a safe fallback. With so many available,
written with sane defaults, users will rarely need to mess around with manual
editing and compiling.

The full base install of Slackware includes many of the useful command-line
tools and development tools that end up getting installed gradually on typical
systems anyway. The 'Full' Slackware install is recommended, as it includes many
of the development files and libraries needed for building and installing
software from Slackbuilds. On the rare occasion when no Slackbuild is available,
=src2pkg= helps make Slackware packages from any source.

=sbopkg= makes installing Slackbuilds easy. It detects user modifications to the
scripts, and offers a choice between using the original or edited version. This
prompt helps to remind the user of any previous changes they made. Furthermore,
any enabled options only apply to that Slackbuild, with no knock-on effect, so
it is easy to keep track of exactly what behaviour has changed on the system.

Slackbuilds use simple, but well documented shell scripts. This is a refreshing
change from the confusing syntactic sugar and complex layers of abstraction
found in the installation scripts of many build systems. Flags listed at the
start of the script make it easy for the user to discover and set the
interesting features of software.

The combination of queue files and Slackbuilds means users only have to
intervene with program installation on rare occasions. =sbopkg= can use [[http://www.sbopkg.org/queues.php][queue]]
files to find the right Slackbuilds and install them as dependencies in the
correct order. =sqg= tool is used to keep an up-to-date list of queue files for
all Slackbuilds.

When updating a Slackware installation following the stable branch, security
patches and newer versions of programs are installed, but there are never any
drastic changes between releases. The base system mostly remains the same, and
by sticking to Slackbuilds, or well vetted third-party packages (such as those
from the Alien BOB repos), breakage is unlikely.

Furthermore, the amount of third-party tools available for converting to
Slackware packages is impressive (.deb, .rpm, Perl, Python, Node.js, Ruby, etc).

So for a stable system that you can rely on for daily work and play, take a look
at Slackware.
