---
layout: post
published: false
title: Finding a stacking window manager for optimal trackpoint usage
---

All window managers should support:

 - click-to-raise and/or click-to-focus (preferably both)
 - click to close, move, and resize
 
Unforunately, however, there seems to be a cock waving excercise amongst the Linux "elite" over who can design a window manager that supports the least mouse usage possible. This is silly, as the mouse is preferable in certain situations (assuming you don't have a handicap), and this is an objective fact. In this post, I will explain how I can make such a bold claim.  

Anyone who says that they can't think of even one use case for preferring the mouse over a keyboard is either not trying, or being dishonest. Hold a drink in one hand and try to copy some text from your web browser to a terminal using just the keyboard. Even assuming you have a keyboard-optimised browsing extension enabled, you still need to either move the caret to select text, or search for the start point of the text, create a mark, and search for the end point. This will also require switching modes if using vim-like keybindings, or using finger gymnastics if using emacs-like keybindings with one hand (remember you are holding a drink). Either way, this generates a lot of mental activity that frustrates ones train of thought, whereas just selecting the text with a mouse and middle clicking in the terminal to paste is an almost subconscious action, one that only requires to pin-point two spacial targets.

You might consider the above argument a cop-out because I'm using the very specific case of holding a drink in one hand, but anyone who says they use a computer and always has two hands on the keyboard whilst working is lying. No one is stooped over a computer desk 100% of the time. Oft-times they will be passing documents, leaning on their elbow on a laptop, stroking their neck-beard, jerking off, or 100 other reasons. So why limit yourself to the keyboard when it is very easy to add an xlib/xcb call to enable some basic pointer function if your window manager? Please stop designing arcane window management just for the sake of geek-cred.
