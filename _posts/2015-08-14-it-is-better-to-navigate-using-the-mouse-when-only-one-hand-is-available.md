---
layout: post
published: false
title: It is better to navigate using the mouse when only one hand is available
---

A great deal of casual computing tasks are easily, and, in fact, more comfortably done with only one hand on the keyboard, trackpoint, or mouse/trackball. For instance, I can happily go for an hour clicking links on reddit with my Thinkpad Trackpoint, without needing to reach for the keyboard at all. If I'm on the couch, this means I can slouch out and rest on one elbow, or have a cup of coffee in one hand. The idea that people should use their computers with two hands over the keyboard at all times, without exception, is an asinane one, but one that seems implicit in todays "mouse is evil" Linux culture. Code monkeys perhaps do go for long periods of times not needing to reach for their mouse, but I feel that even the most hardcore developer is kidding themselves in saying that there aren't periods of time when using a mouse would be easier.

I've tried to play devils advocate with myself and not use the trackpoint (or touchpad) on my Thinkpad for a week, but operating window manager shortcuts with one hand is just plain awkward, however elaborate the keybinding setup. Window manager keyboard shortcuts inherantly require a modifier key (e.g., holding onto 'Alt' key or 'Windows' key, and will do so until the day when someone designs advanced AI that can contextually detect when someone is typing, rather than performing a management action on the window frame itself. The problem with needing to hold a modifier key, however, is that there are plenty of times when only one hand is rested on the keyboard.  

Examples of such situations:

- holding a drink
- passing documents
- leaning your elbow on the couch (if on a laptop)
- picking your nose
- stroking your neck beard

Anyone who says they never do any of the above is lying; nobody is stooped over a computer desk with both hands on the keyboard 100% of the time. 

With only one hand on the keyboard, a key map is needed that prevents too much travel across the keyboard, and prevents finger gymnastics to press the right keyboard combinations. 

But how is such a keymap achieved? Hypothetically, I could bind all my window manager binds to the right side of the keyboard; that way, when I'm trying to use my computer with one hand, I don't need to keep reaching across the keyboard to hit the enter key, or various key combos. I'm yet to find a keyboard, however, without ergonomical problems that spoil this idea, namely:

 - the placement of right ctrl on most keyboards is hard to discover and usually requires a pinky stretch (especially on laptops where it is squashed in by the arrow keys)
 - shift is practically unreachable as a modifier, and better served as a replacement for CapsLock when typing single-handedly
 - alt key is reachable with the thumb, but most applications use it on the top-level, so it is inflexible as a modifier for window management keys

And even if there is a keyboard out there that works well with right handed key bindings, I can't guarantee I'll have access to such a keyboard across computers. On the left side of the keyboard the problems I've listed, in the main, don't exist. I can remap CapsLock as a Ctrl key or use the Win key as a modifier, both of which are easily discoverable. Even so, one handed usage still requires travelling right across the keyboard to press Enter. I've considered rebinding Enter to e.g., Ctrl, but this, again, isn't a very portable solution, and is probably asking for disaster in a terminal. 

Try doing any of the above tasks whilst also copying some text from your web browser to a terminal using just the keyboard. Even assuming you have a keyboard-optimised browsing extension enabled, you still need to either move the caret to select text, or search for the start point of the text, create a mark, and search for the end point. This will also require switching modes if using vim-like keybindings, or using finger gymnastics if using emacs-like keybindings (remember you only have one hand available). Either way, this generates a lot of mental activity that frustrates ones train of thought, whereas just selecting the text with a mouse is an almost subconscious action, and one that only requires to pin-point a spacial target.

All window managers should at least optionally support:

 - click-to-raise (and/or clicking on titlebar and/or window borders to raise)
 - click to close, move, and resize
 
Unfortunately, however, there seems to be a cock waving excercise amongst the Linux "elite" over who can design a window manager that supports the least mouse usage possible. This is sad, considering that there is little cost or difficulty for WM developers to do the kind thing by adding a couple of lines of xlib or xcb calls to enable some basic mouse functionality for those who don't take such a hard line on mouse usage. Please stop designing arcane window management just for the sake of geek-cred.

I wouldn't even be that arsed to make a whole post about this, but I've noticed one too many websites and forum posts from developers listing "no mouse" as a feature of the WM. Being forced to hold onto MOD key + trackpoint on my Thinkpad just to raise a window when I'm using one hand makes your window manager automatically crap. Fair enough, you gave up your time to make it for free, and no one forced me to waste 10 minutes of my life installing and quickly deleting it, but  stop spamming forums and encouraging your band of fellow cock-wavers with propoganda about the evils of mouse usage; it is academically dishonest. You fucking know you want to reach for that mouse. You miss it. Stop lying to yourself. 


