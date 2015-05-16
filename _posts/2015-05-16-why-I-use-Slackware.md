---
layout: post
published: true
title: Why I use Slackware
---



# Why would I want to use Slackware?

To put it simply: it is stable, simple to use, and sane. However, "stable", "simple" and "sane" are subjective terms, so allow me to elaborate. 

Slackware is great for people like me who are OCD over keeping full control of their system. It requires you to learn a few standard practices and conventions, and maybe even keep a few notes about the changes you've made, but at least you will understand how things interrelate on your system. Slackware does offer some automated tools and scripts for repetitive tasks, but also encourages more user interaction and understanding than most other distros do. To me this is a good thing, because automated tools might not do what you think they're doing, and in the long term often lead to undesirable changes and untraceable breakage. 

The thing that puts many off using Slackware is that it has no officially supported dependency resolution for its packages. I've found, however, that in practice dependency conflicts never happen on Slackware, because 90% of the software I need has a `Slackbuild` available already. Slackbuilds tend to use other Slackbuilds as dependencies, or the base install itself, so nothing conflicts. 

`sbopkg` makes installing Slackbuilds easy. It will detect modifications you make to the Slackbuilds and offer a choice between using the original or edited version. This prompt gives me peace-of-mind that I won't forget about changes I've made. Any options set only apply to that Slackbuild, and will not change the defaults for other Slackbuilds, which is reassuring to know for a control freak such as myself.

With some other distros I've used, I found the build scripts a bit overwhelming to parse and understand. They used their own syntactic sugar -- sugar that admittedly probably helps package maintainers who do a lot of packaging -- but it just seems to add unnecessary layers of complexity for the average user. Conversely, Slackbuilds use simple, standard shell script. They are well documented and give some pointers about what options you might want to set for that particular software (often using shell variables to make toggling these options easier). This is my idea of simplicity.

There is no need to mess about with compile time options and dependencies if you don't want to either. `sbopkg` supports `queue` files to find and install all the needed dependencies automatically. So if you don't care about installing all the possible dependencies for a package, sbopkg will offer to use a queue file to install everything in the right order first time. I've never had a problem installing a Slackbuild in this manner. 

Also, the full base install of Slack includes many of the useful command line tools and development tools one might gradually end up installing anyway. Only on very rare occasions have I needed to compile a package manually, and even then there are tools to make Slack packages from source such as `src2pkg`. 

When updating a stable Slackware installation, you still get security patches and newer versions of programs, such as Firefox, but there are never any drastic changes. The base system mostly remains the same, and if you stick to using Slackbuilds or well vetted third-party packages (such as those from the Alien BOB repos) breakage is unlikely.

So if you want a system that you can tinker with as you please, but still actually rely on for daily work and play, take a look at Slackware 14.1-Stable. 
