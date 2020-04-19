---
title: "Mac International keyboard layout in Gnome"
date: 2020-04-19T10:50:52+02:00
lastmod: 2020-04-19T10:50:52+02:00
draft: false
keywords: [xkbd]
description: "How to setup ISO Level 3 to any modifier"
tags: [xkbd, gnome, ubuntu]
categories: []
author: "Vlad Vasiliu"

toc: false
postMetaInFooter: true
hiddenFromHomePage: false

---

For international users the MacOS layout allows using a QWERTY keyboard but also entering local symbols. This is not directly available on Gnome.

<!--more-->

## Goal

I'm usig regularly both a Mac and Linux PC. I prefer the US layout for programming use but I also have to type in French.
The standard US layout on the Mac is perfect for this as it allows me to use the familiar QWERTY layout for system / programming tasks but I'm also able to type in French symbols easily.

What I'm looking for is to be able to use the same layout on the Linux box.
For that, I'm attempting to configure `Win` and / or `ALT` to act as Level 3 Shift.
I'm using `Left Win` and `Right ALT` on my keyboard as they are the closest to the location on a Mac keyboard.


## Environment

This is tested on Ubuntu 20.04 with Gnome 3.34.


## Quick steps

1. In *Settings* -> *Region and language* -> *Input sources* select **English (Macintosh)**.
2. Open a terminal to reconfigure xkb for using `Right ALT` and `Left WIN` for Level3 Switch. Adjust as necesary.

```bash{linenos=false}
dconf write /org/gnome/desktop/input-sources/xkb-options "['lv3:ralt_switch', 'lv3:lwin_switch']"
```

## Explanation

The layout to be used is US Mac, which allows using level 3 Symbols for composing some characters, such as **œ** (`alt` + `q`) and **é** (`alt` + `e`, `e`).

This sets up the right ALT key to behave like the Mac ALT key, that is ISO Level 3 Shift. This may or may not work for you depending on the physical layout of the keyboard.

There is no way to configure the left hand side modifier directly through Gnome Settings.
Gnome also seems to ignore XKB configuration set up via `localectl` for example.
The solution is to set up the configuration on the command line. See point two above.


## References

* [Arch Wiki • X keyboard extension](https://wiki.archlinux.org/index.php/X_keyboard_extension)
* [Arch Wiki • Keyboard configuration](https://wiki.archlinux.org/index.php/Xorg/Keyboard_configuration#Setting_keyboard_layout)
