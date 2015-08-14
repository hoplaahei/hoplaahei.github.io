---
layout: post
published: false
title: Finding a stacking window manager for optimal trackpoint usage
---

All window managers should at least optionally support:

 - click-to-raise (and/or clicking on titlebar and/or window borders to raise)
 - click to close, move, and resize
 
Unfortunately, however, there seems to be a cock waving excercise amongst the Linux "elite" over who can design a window manager that supports the least mouse usage possible. This is silly, as the mouse is preferable in certain situations (assuming you don't have a handicap), and this is an objective fact. Allow me to explain this bold claim. 

I'll agree that the keyboard is theoretically just as quick for most tasks as the mouse when two hands are placed over the keys, but the fact is that there are plenty of times when two hands do not rest on the keyboard. Examples of such situations:

- holding a drink
- passing documents
- leaning your elbow on the couch (if on a laptop)
- picking your nose
- jerking off
- stroking your neck beard
- 
Anyone who says they never do any of the above is lying; nobody is stooped over a computer desk with both hands on the keyboard 100% of the time. Try doing any of the above tasks whilst also copying some text from your web browser to a terminal using just the keyboard. Even assuming you have a keyboard-optimised browsing extension enabled, you still need to either move the caret to select text, or search for the start point of the text, create a mark, and search for the end point. This will also require switching modes if using vim-like keybindings, or using finger gymnastics if using emacs-like keybindings (remember you only have one hand available). Either way, this generates a lot of mental activity that frustrates ones train of thought, whereas just selecting the text with a mouse is an almost subconscious action, and one that only requires to pin-point a spacial target.

So why limit yourself, or others, to the keyboard when there is little cost or difficulty to adding a couple of lines of xlib or xcb calls that enable some basic mouse functionality in your window manager? Please stop designing arcane window management just for the sake of geek-cred.

I wouldn't even be that arsed to make a whole post about this, but I've noticed one to many websites and forum posts from developers listing "no mouse" as a feature of the WM. Being forced to hold onto MOD key + trackpoint just to raise a window when I'm using one hand makes your window manager automatically crap. Fair enough, you gave up your time to make it for free, and no one forced me to waste 10 minutes of my life installing and quickly deleting it, but  stop spamming forums and encouraging your band of fellow cock-wavers with propoganda about the evils of mouse usage; it is academically dishonest. You fucking know you want to reach for that mouse. You miss it, especially when jerking it to porn. Stop lying to yourself. 


