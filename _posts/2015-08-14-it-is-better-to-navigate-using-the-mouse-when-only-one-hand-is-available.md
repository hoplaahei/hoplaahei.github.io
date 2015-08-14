---
layout: post
published: false
title: It is better to navigate using the mouse when only one hand is available
---

Window manager keyboard shortcuts inherantly require a modifier key (e.g., holding onto 'Alt' key or 'Windows' key, and will do so until the day when someone designs advanced AI that can contextually detect when someone is typing rather than performing a management action on the window frame itself. The problem with needing to hold a modifier key, however, is that there are plenty of times when only one hand is rested on the keyboard; this requires finger gymnastics to press the right keyboard combinations. Examples of such situations:

- holding a drink
- passing documents
- leaning your elbow on the couch (if on a laptop)
- picking your nose
- jerking off
- stroking your neck beard

Anyone who says they never do any of the above is lying; nobody is stooped over a computer desk with both hands on the keyboard 100% of the time. Hypothetically, I could bind all my window manager binds to the right side of the keyboard; that way, I don't need to keep reaching across the keyboard to press window manager shortcuts or enter key whilst typing with one hand. The problem with this, however, is that it requires finger gymnastics of the pinky finger to stretch for the modifier keys on any keyboard. Furthermore, myself, and the majority of people, are right hand dominant, and it feels awkward to use the left hand for other tasks, such as holding a cup of coffee. I've considered rebinding the enter key to the left side of my keyboard, but that is asking for disaster when in a terminal, and not a very portable solution across different setups. 

Try doing any of the above tasks whilst also copying some text from your web browser to a terminal using just the keyboard. Even assuming you have a keyboard-optimised browsing extension enabled, you still need to either move the caret to select text, or search for the start point of the text, create a mark, and search for the end point. This will also require switching modes if using vim-like keybindings, or using finger gymnastics if using emacs-like keybindings (remember you only have one hand available). Either way, this generates a lot of mental activity that frustrates ones train of thought, whereas just selecting the text with a mouse is an almost subconscious action, and one that only requires to pin-point a spacial target.

All window managers should at least optionally support:

 - click-to-raise (and/or clicking on titlebar and/or window borders to raise)
 - click to close, move, and resize
 
Unfortunately, however, there seems to be a cock waving excercise amongst the Linux "elite" over who can design a window manager that supports the least mouse usage possible. This is sad, considering that there is little cost or difficulty for WM developers to do the kind thing by adding a couple of lines of xlib or xcb calls to enable some basic mouse functionality for those who don't take such a hard line on mouse usage. Please stop designing arcane window management just for the sake of geek-cred.

I wouldn't even be that arsed to make a whole post about this, but I've noticed one too many websites and forum posts from developers listing "no mouse" as a feature of the WM. Being forced to hold onto MOD key + trackpoint on my Thinkpad just to raise a window when I'm using one hand makes your window manager automatically crap. Fair enough, you gave up your time to make it for free, and no one forced me to waste 10 minutes of my life installing and quickly deleting it, but  stop spamming forums and encouraging your band of fellow cock-wavers with propoganda about the evils of mouse usage; it is academically dishonest. You fucking know you want to reach for that mouse. You miss it, especially when jerking it to porn. Stop lying to yourself. 

