---
title: "Option (&#x2325;) keyboard shortcuts for Terminal and iTerm"
layout: 'post'
author: bo
---

Throughout Mac OS X, almost every input field and text area have
consistent keyboard shortcuts to navigate and edit the text. However, in
both Terminal and iTerm, some of these don’t work like you might be used
to. Furthermore, both iTerm and Terminal respond differently to each
other, as well as to hour the rest of the system does it.

Three shortcuts I’ve especially missed are: Option-Left to move left one
word, Option-Right to move right one word, and Option-Backspace to
delete one word to the left.

The ideal solution to get these working as I’d like would be to
configure an `inputrc` file. However, after playing around for a few
hours, I still couldn’t get this working reliably using the Option key.
So, my fallback is to configure the terminal apps to do this themselves.

If you set up the terminal applications to use Option as the Meta key,
some remote systems will already work with the keyboard shortcuts I
want. However, locally only the Option-Backspace one works and even then
it only works out of the box in Terminal.

So, here’s how to get all three working in both Terminal and iTerm,
locally and on remote systems:

## Terminal

First, enable Option as meta key option by going to the *Preferences \>
Settings \> Your Profile \> Keyboard( and checking “Use option as meta
key” check box. This will get Option-Backspace working.

Next, while still in the Keyboard settings from the last step, press the
+ button. In the popup, choose “cursor left” for the Key, “option” for
the Modifier, and “send string to shell:” for the Action. In the text
field, type Esc then lowercase B key and press OK. This will get the
Option-Left shortcut working.

To enable the Option-Right shortcut, repeat the previous step, instead
choosing “cursor right” for the Key, and typing Esc then lowercase F in
the text field.

If you switch Terminal themes, you will have to repeat these steps for
the new theme.

## iTerm

For the iTerm configuration, I will simply point you to [this excellent
article](http://greyworld.net/en/blog/iterm-word-shortcuts/) which help
me get up and running. The sections that correspond to the shortcuts I
set up are 2.1, 2.2, and 2.4.

