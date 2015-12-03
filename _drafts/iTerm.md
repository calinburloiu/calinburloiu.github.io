---
layout: post
title:  Setting iTerm2 Shortcuts to be Consistent with Other Mac Shortcuts
---
Skeleton
--------

All configuration performed in "Preferences...".

By default the preferences are saved in
"/Users/burloiu/Library/Preferences/com.googlecode.iterm2.plist". You can change this from General by ticking "Load preferences from a custom folder or URL" and selecting a directory to load (if you already have) or save (if saving a new one) the configuration from.

### Keys -> Global Shortcut Keys

Set "Left option key acts as" "+Esc".

* Shift + Command + { -> Previous Tab
* Shift + Command + } -> Next Tab
* Option + Shift + Command + { -> Move Tab Left
* Option + Shift + Command + } -> Move Tab Right
* Command + Up -> Scroll To Top
* Command + Home (fn + Left) -> Scroll To Top
* Command + Down -> Scroll To End
* Command + End (fn + Right) -> Scroll To End
* Page Up (fn + Up) -> Scroll One Page Up
* Shift + Page Up (fn + Up) -> Scroll One Page Up
    - PC style
* Page Down (fn + Down) -> Scroll One Page Down
* Shift + Page Down (fn + Down) -> Scroll One Page Down
    - PC style
* Control + Tab -> Cycle Tabs Forward

Not Mac OS X shortcuts, but I assigned a shortcut to them:

* Option + Up -> Scroll One Line Up
* Option + Down -> Scroll One Line Down

#### Backup

What do these things do? Should I care about them?

* Option + Up -> Send Hex Codes "0x1b 0x1b 0x5b 0x41"
* Option + Down -> Send Hex Codes "0x1b 0x1b 0x5b 0x42"

### Profiles -> Keys -> Profile Shortcut Keys

* Option + Left -> Move Cursor One Word Left -> Send Escape Sequence + b
* Option + Right -> Move Cursor One Word Right -> Send Escape Sequence + f
* Command + Left -> Move Cursor At Beginning Of Line -> Send Escape Sequence + 1
* Command + Right -> Move Cursor At End Of Line -> Send Escape Sequence + 5
* Option + Delete -> Delete Word To The Left Of Cursor -> Send Escape Sequence + 17

