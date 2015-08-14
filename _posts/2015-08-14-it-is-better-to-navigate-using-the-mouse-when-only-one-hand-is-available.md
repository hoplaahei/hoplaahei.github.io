---
layout: post
published: false
title: "Lite-weight" window manager developers: stop making it hard to use a computer with one hand
---

I wouldn't even be arsed to make a whole post about this, but in my quest to find a portable and unbloated window manager, I've noticed one too many websites and forum posts from developers taking great pride in the fact that their window manager has limited to no mouse support at all. Look matey, being forced to hold onto MOD key + trackpoint on my laptop just to raise a window when I'm using one hand makes your window manager crap. Fair enough, you gave up your time to make it for free, and no one forced me to waste 10 minutes of my life installing and quickly deleting it, but please stop trying to push the arcane and academically dishonest idea that mouse usage is better off eradicated from a computer work-flow. 

I'm going to argue that there is a specific use case where a pointing device objectively excels over the keyboard: one-handed use. First, I want to outline why one handed operation is the norm rather than the exception for many people's computer workflows. Some examples of when it is comfortabler for me to only operate a laptop with one hand are when I'm:

-  flicking through physical documents
-  cuddling my girfriend
-  leaning on my elbow on the couch
-  having a cup of coffee
-  eating
-  scratching my neckbeard

I could go on, but hopefully this highlights the fact that for casual usage (i.e., most of the populace) having both hands at the ready at all times when operating a computer is impractical. Sadly, there is a trend of "lightweight" window managers to make it very hard to operate with one hand because of one simple ommission: they do not allow a "click-to-raise" policy. This means that my casual web browsing is completely fucked, as I now have to pinky stretch to a modifier key just to e.g., raise my web browser back above the terminal when I'm pasting code. 

I've tried to play devils advocate with myself and not use the trackpoint (or touchpad) on my Thinkpad for a week, but operating window manager shortcuts with one hand is just plain awkward, however elaborate the keybinding setup. Window manager keyboard shortcuts inherantly require a modifier key (e.g., holding onto 'Alt' key or 'Windows' key, and will do so until the day when someone designs advanced AI that can contextually detect when someone is typing, rather than performing a management action on the window frame itself. The problem with needing to hold a modifier key, however, is that there are plenty of times when only one hand is rested on the keyboard.  

With only one hand on the keyboard, a key map is needed that prevents too much travel across the keyboard, and prevents finger gymnastics to press the right keyboard combinations. 

But how is such a keymap achieved? Hypothetically, I could bind all my window manager binds to the right side of the keyboard; that way, when I'm trying to use my computer with one hand, I don't need to keep reaching across the keyboard to hit the enter key, or various key combos. I'm yet to find a keyboard, however, without ergonomical problems that spoil this idea, namely:

 - the placement of right ctrl on most keyboards is hard to discover and usually requires a pinky stretch (especially on laptops where it is squashed in by the arrow keys)
 - shift is practically unreachable as a modifier, and better served as a replacement for CapsLock when typing single-handedly
 - alt key is reachable with the thumb, but most applications use it on the top-level, so it is inflexible as a modifier for window management keys

And even if there is a keyboard out there that works well with right handed key bindings, I can't guarantee I'll have access to such a keyboard across computers. On the left side of the keyboard the problems I've listed, in the main, don't exist. I can remap CapsLock as a Ctrl key or use the Win key as a modifier, both of which are easily discoverable. Even so, one handed usage still requires travelling right across the keyboard to press Enter. I've considered rebinding Enter to e.g., Ctrl, but this, again, isn't a very portable solution, and is probably asking for disaster in a terminal. 

Try doing any of the above tasks whilst also copying some text from your web browser to a terminal using just the keyboard. Even assuming you have a keyboard-optimised browsing extension enabled, you still need to either move the caret to select text, or search for the start point of the text, create a mark, and search for the end point. This will also require switching modes if using vim-like keybindings, or using finger gymnastics if using emacs-like keybindings (remember you only have one hand available). Either way, this generates a lot of mental activity that frustrates ones train of thought, whereas just selecting the text with a mouse is an almost subconscious action, and one that only requires to pin-point a spacial target.

In my quest to find a window manager, I found so many Github issues raised asking for mouse support. And yet, many developers take a very hard line and deny any such requests, often touting the lack of mouse support as a feature of their window manager. Refusing to add such basic support is silly; it involves little cost or difficulty to add a couple of lines of xlib or xcb calls that introduce a click-to-raise configuration option for those users who want it. 
The tl;dr is that all window managers should at least optionally support:

 - click-to-raise (and/or clicking on titlebar and/or window borders to raise)
 - click to close, move, and resize
