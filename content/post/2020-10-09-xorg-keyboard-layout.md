---
layout: post
date: 2020-10-09
title: Keyboard layout in xorg
tags: ["xorg"]
---

In this post, we will look at how to add a second keyboard layout in under Xorg and set up a keyboard shortcut to switch between them. We will also see how to remap keys.


You can check the current keyboard layout used by the xorg by running **setxkbmap -query**. You can use the same command to modify the layouts and options by using -layout or -option switches. As mentioned in manual, -option will append the passed in arguments to already set options. The alternative is to create a xorg configuration file and place it in xorg.conf directory. But how do we know what layout to chose or what options to set?

Under OpenBSD, **/usr/X11R6/share/X11/xkb/** folder holds various xkb related files including layout definitions and options. Of the many folders, the one that is useful for our purpose is symbols. Here you will find files with names mostly matching the country codes in addition to some special purpose files. For example, the US keyboard definition is **us** file whereas the Indian language layouts are defined in **in** file. Also it should be noted that these files may themselves include other files. For example, there is a separate file for Indian currency called **rupeesign** which is then included in **in** file.

For illustration, let us add **devangari** layout in addition to the existing **us** layout. Devanagari being an Indian language, I expect to find its name in the **in** file. When I look into the file, I see the layout is named as **deva**. The next task will be to find out a way to switch the layouts.

If you noticed earlier, there was folder called **keycodes** alongside **symbols** folder. This contains mapping for keyboard scan codes to a symbolic name. The types folder defines how the keycodes are changed when the key is pressed along with a modifier key such as shift. For example, pressing a key with shift pressed will produce a different symbol than without it. The **compat** folder will define behaviors of modifiers and led indicators. Geometry may be used to display the keyboard image. Using the **rules** you can define your own custom mapping by creating a rule file.

A level represents the sort of thing that a shift key is expected to do. Normally, when you press the key marked A you would expect an a character to appear. If you hold the shift key down and press the same key, you would expect an A to appear instead. The basic idea behind groups is that, sometimes, you want to shift the entire keyboard over to some other character set, so that you can access characters not usually considered part of a standard keyboard, such as , , or . 
An example below shows a keycode with four symbols mapped. Pressing it without any modifiers gives symbol **c**, pressing with shift will give **C**, using right alt produces copyright symbol whereas a combination of right alt, shift and the key will produce cent key. 
```
    key <AB03> { [         c,          C,     copyright,             cent ] };
```

The **group** file in the symbols table defines various keyboard shortcuts to switch the groups. I am using a combination of Alt+Shift to switch the groups. Create a file called layout.conf with below contents and place it in /etc/X11/xorg.conf.d directory. If you don't have the directory, create one and place the file.

```
Section "InputClass"
	Identifier "system-keyboard"
	MatchIsKeyboard "on"
	Option "XkbLayout" "us,in"
	Option "XkbVariant" "basic,deva"
	Option "XkbOptions" "grp:alt_shift_toggle,altwin:prtsc_rwin"
EndSection
```

Once you restart X, you should be able to switch the layouts by pressing Shift+Alt together. If you noticed, I have used one more option called **altwin** that remaps the print screen key between right alt and control to super key. You can go through different files in the symbols folder to find out more such mappings. For example, capslock file maps capslock to different symbols.

## Reference
1. [How to further enhance XKB configuration](https://www.x.org/releases/X11R7.0/doc/html/XKB-Enhancing.html)
2. [An Unreliable Guide to XKB Configuration](https://www.charvolant.org/doug/xkb/html/xkb.html)
