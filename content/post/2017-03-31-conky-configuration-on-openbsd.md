---
layout: post
date: 2017-03-31
title: Conky Configuration On OpenBSD
tags: ["conky"]
---

Conky is a lightweight system monitoring tool that will help monitor the system
parameters. I will explain the configuration as used on my OpenBSD machine.

## Installation
Run the pkg_add to install the conky
```
	pkg_add conky
```
Choose the version with no xmms2 and with x11

Add the line for conky in the .xinitrc file in your home directory to start it
in background.
```
	conky &
```
If you start X, your should see a default conky window.

## Basics
Conky configuration file is split into two sections. One before the "TEXT" line
and one after it. The first section is used to define default values and set the
configuration. The second section is what decides what is shown on the sceen.

## Configurations
Set up the default font color for the text.
```
	default_color green
```

Use an anti aliased font by using xft and set the default font size. To get the
list of fonts available on your system, run the fc-list command and the
choose the font that fits best for you.
```
	use_xft yes
	xftfont DejaVu Sans Mono:size=9:weight=Bold
	xftalpha 0.5
```
Avoid the screen flickering by setting the double buffer. This way the screen is
drawn in memory and displayed at once rather than partially.
```
	double_buffer yes
```
Display conky in its own window. And set the type as override. When you are
alt-tabbing through the windows, conky window will not be selected by marking it
override. Other possible types are "desktop", "panel", "dock". Refer to man page
for more details.
```
	own_window yes
	own_window_type override
```
Set the alignment to top middle. This will display conky at the top center. We
also specify its width as 1366 to cover entire screen width. This value should
be changed as per your screen resolution. You can get your screen dimension by
running, "xdpyinfo|grep dimension"
```
	alignment top_middle
	minimum_size 1366
```
You can disable the border around the window set to refresh conky every second.
```
	draw_borders no
	update_interval 1
```
Now set a 5 pixel gap from the top so that your text is not hidden by the screen
border.
```
	gap_x 0
	gap_y 5
```
Some of the variables like network speed change their values drastically which
causes the other text to dance around. Using the spacer makes such variables
fixed width.
```
	use_spacer left
```

## Text section
The text below is what is displayed exactly on the window. This configuration
displays entire text in one line. Backslash followed by newline acts as a line
continuation.
The section begins with the line "TEXT" followed by the variable names.
```
	TEXT
	CPU Load: ${loadavg} | \
	Freq: $freq_g GHz | \
	RAM: $memperc% | \
	Uptime: $uptime_short \
	${alignc -250} \
	${color white}${time %T} ${time %a} ${time %v}${color} \
	${alignr 15} \
	Vol: ${exec mixerctl outputs.master | cut -d "=" -f2 | cut -d "," -f1} | \
	Mute: ${exec mixerctl outputs.master.mute | cut -d "=" -f2 } | \
	Down: ${downspeed urtwn0} Up: ${upspeed urtwn0} | \
	Battery: $apm_battery_life $apm_battery_time $apm_adapter
```
Each variables values can be accessed with a $. It can enclosed in a brace if
you have more than one words for example to pass a parameter. Any other text
will be displayed literally on screen.
Most of the variables are self explanatory. The alignc marks the following text
to be center aligned. Passing a number to it shifts the text that many pixels to
the left. Passing a negative number shifts it to right. This value should be
adjusted based on trial and error on your screen.

alignr is similarly aligns following text to the right. A value of 15, leaves a
gap of 15 pixels to the right.

The ${color white} shows the following text in white until it hits another color
variable. A color without any color name sets the default color which was
defined in the configuration section.

The time format can be adjusted by passing a conversion specification. You can
refer the man page of strftime(3) to understand various available formats.

$exec is a special variable in that it executes its parameters on the host. This
can be used to get the values from your system and plug it to the conky window.

Both downspeed and upspeed should be passed with a network adapter name. This
can be em0,, athn0 etc based on what network interface you are using. Running an
ifconfig(8) can help you find that out.

## Reference
1. [conky wiki](https://github.com/brndnmtthws/conky/wiki)
