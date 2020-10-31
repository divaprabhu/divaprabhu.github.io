---
layout: post
date: 2016-11-05
title: Configuring Touchpad With Synaptics
tags: ["synaptics", "touchpad"]
---

Create a configuration file called xorg.conf with below contents and place it in
/etc/X11/

```
Section "InputClass"
    Identifier "touchpad"
    Driver "synaptics"
    MatchIsTouchpad "true"  
    Option "TapButton1" "1"
    Option "TapButton2" "3"
    Option "TapButton3" "2"
    Option "VertTwoFingerScroll" "on"
    Option "HorizTwoFingerScroll" "on"
EndSection
```

The TapButton1 to TapButton3 indicate one, two or three finger taps. The
corresponding value of "1","3","2" indicate left click, middle click and right
click respectively.

The next two lines enable horizontal and vertical scrolling.
