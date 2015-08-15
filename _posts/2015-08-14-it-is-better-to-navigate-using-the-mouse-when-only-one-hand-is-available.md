---
layout: post
published: false
title: "Lightweight" stacking window manager developers: stop making it hard to use a computer with one hand
---

Firstly, I'm not going to go into the reasons why tiling window managers don't suit my workflow, as it will only take away from the focus of this post, which is about stacking window managers. Secondly, I wouldn't even be arsed to make a whole post about this, but in my quest to find a portable and unbloated stacking window manager, it pissed me off to see so many proclamations of window-manager devs on websites and forum posts taking great pride in the fact that their window manager has limited to no mouse support. Look, being forced to hold onto MOD key + click on my laptop just to raise a window when I'm using one hand relegates your window manager to the usability levels of the Xerox Park. Fair enough, you gave up your time to make it for free, and no one forced me to waste 10 minutes of my life installing and quickly deleting it, but please stop trying to push the arcane and academically dishonest idea that pointing device usage is better off eradicated completely from a stacking work-flow. 

I'm going to argue that there is a specific use case where a pointing device objectively excels over the keyboard in stacking window management: one-handed use when not doing a lot of typing. First, I want to outline why one handed operation (when not doing a lot of typing) is the norm rather than the exception for many people's computer workflows. Some examples of when it is comfortabler for me to only operate a laptop with one hand are when I'm:

-  flicking through and referencing physical documents
-  cuddling my girfriend
-  leaning on my elbow on the couch
-  having a cup of coffee
-  eating
-  scratching my neckbeard

I.e., most of the time. People who code for their day job may well spend more time stooped over the keyboard with both hands. However, someone who only casually codes, such as myself, will spend only a couple of hours typing intensely, but spend more time browsing the web, researching, and copy and pasting the odd bit of text. Sadly, there is a trend of "simple, usable, lightweight" aka meme WMs to make it very hard to operate with one hand, because they operate under the assumption that everyone is a programmer, who always has two hands on the keyboard. There is a simple code ommission from many such stacking WMs that makes it hard for users to operate with one hand: not allowing click-to-raise policy (clicking anywhere in the visible part of a window to raise it above others). Alternative policies such as sloppy focus or click-to-focus are probably great for someone who spends all day writing code, but the casual user (one who often operates with only one hand on an input device) really finds click-to-raise essential. Otherwise, their web browsing experience is completely fucked, as they now have to pinky stretch to a modifier key just to e.g., switch to a terminal when pasting code. Even if a stacker forgoes click-to-raise, but does support clickable titlebars and borders, it still only offers small and awkward to hit targets, whereas click-to-raise offers the huge targets of whole windows. Remember here: I'm only talking in the context of one-handed usage, where it is downright awkward to frequently snap up to the window edges and click titlebars and borders. 

I've tried to play devils advocate with myself and not use the trackpoint (or touchpad) on my laptop for a week, but operating mod-key bound window manager shortcuts with one hand is just plain awkward, however elaborate the keybinding setup. Top-level WM shortcuts inherantly require a modifier key (e.g., holding onto 'Alt' key or 'Windows' key, and will do so until the day when someone designs advanced AI that can contextually detect when someone is typing, rather than performing a management action on the window frame itself. The problem with needing to hold a modifier key, however, is that, as aforementioned, there are plenty of times when only one hand is rested on the keyboard.  

With only one hand on the keyboard, a key map is needed that prevents too much travel across the keyboard, and one that prevents finger gymnastics to press the right keyboard combinations. But how is such a keymap achieved? Hypothetically, I could bind all my window manager binds to the right side of the keyboard; that way, when I'm trying to use my computer with one hand, I don't need to keep reaching across the keyboard to hit the Enter key. However, I'm yet to find a keyboard without ergonomical problems that make it uncomfortable to perform key-combos, namely:

 - the placement of right ctrl on most keyboards is hard to discover and usually requires a pinky stretch (especially on laptops where it is squashed in by the arrow keys)
 - shift is practically unreachable as a modifier, and better served as a replacement for CapsLock when typing single-handedly
 - right alt key is accessible with the thumb, but the muscle movement to position on it still often misaligns your other fingers with the home row, and most applications use it on the top-level anyway, so Alt is inflexible as a modifier for window management keys

And even if there is a keyboard out there that works well with right handed key bindings (thus allowing shorter travel to the Enter key), I still can't guarantee I'll have access to such a keyboard across computers. Admittedly, on the left side of the keyboard, the problems I've listed don't exist, as I can remap CapsLock as a Ctrl key or use the Win key as a modifier -- both of which are easily discoverable. Even so, one handed usage still requires travelling right across the keyboard to press Enter, whereas when operating e.g., a trackpoint, I'm always close to the enter key, and don't need to use modifiers for basic and frequent operations, such as copy, paste, drag and move of windows and objects. I've considered rebinding Enter to e.g., Ctrl, but this, again, isn't a very portable solution, and is probably asking for disaster in a terminal. 

As an example, take the common task of copying some text from your web browser to a terminal using just the keyboard. Even assuming you have a keyboard-optimised browsing extension enabled, you still need to either move the caret to select text, or search for the start point of the text, create a mark, and search for the end point. This will also require switching modes if using vim-like keybindings, or using finger gymnastics if using emacs-like keybindings (remember you only have one hand available). Either way, this generates a lot of mental activity that frustrates ones train of thought, whereas just selecting the text with a mouse is an almost subconscious action, and one that only requires to pin-point a spacial target.

The tl;dr is that all stacking window managers should at least optionally support:

 - click-to-raise (and/or clicking on titlebar and/or window borders to raise)
 - rudimentary titlebar support (with click to close, move, and resize)

I'd make similar arguments for tiling window managers; most of them don't do enough to at least offer some basic mouse support for some edge cases where mouse usage is optimal. Sadly, that would take a whole other post entirely to justify myself, so I'm not going to bother getting into it. 
