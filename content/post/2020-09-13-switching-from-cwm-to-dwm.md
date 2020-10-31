---
layout: post
date: 2020-09-13
title: Switching From cwm To dwm
tags: ["suckless"]
---

It has been a long time since my last post. Since that time I have switched from a cwm user to suckless dwm. This blog outlines the compilation and install procedure for dwm.


[dwm](https://dwm.suckless.org/) is window manager from suckless that focuses on simple, clear and minimal code. Even though I have been a happy user of cwm in the openbsd base, I feel the tiling wm like dwm suites more to my workflow. Moreover, it comes with a built-in panel at the top which is an added bonus.

## Installation
I use the github version with no additional patches. I maintain my own version of patches for dwm, st and dmenu with minimal configuration changes in github. Installation is simply a matter of cloning the repo, applying patches and running make install.

```
export OS_TYPE=$(uname -s)
suckless_common() {
        program=$1
        if [ -d $program ];
        then
                cd $program
                git pull
        else
                git clone git@github.com:divaprabhu/$program.git
                cd $program
        fi
        if [ "$OS_TYPE" = "OpenBSD" ];
        then
		export SUDO="doas"
                patch <patches/openbsd.diff
        elif [ "$OS_TYPE" = "Linux" ];
        then
		export SUDO="sudo"
                patch <patches/linux.diff
        fi

        make && $SUDO make install && make clean && rm config.h

        git checkout -- config.def.h config.mk
        rm config.def.h.orig config.mk.orig 2>/dev/null

        cd ..
}

if [ "$OS_TYPE" = "OpenBSD" ];
then
	cp /etc/examples/man.conf .
	echo "manpath /usr/local/share/man" >> man.conf
	$SUDO mv man.conf /etc/
fi
suckless_common dwm
suckless_common dmenu
suckless_common st
```

This simple scripts installs dwm, dmenu and st terminal from suckless.org. The patches make very simple changes to the stock dwm. The default modkey is set to Alt, but I prefer the Superkey for window manager shortcuts so that Alt can be freed up for application shortcuts. This is done by defining the MODKEY as Mod4Mask instead of the default Mod1Mask.

Two other changes include defining and adding short cut for tmux and screenlock which are set to Super+Ctrl+Return and Super+L respectively

Man pages for these tools are installed to /usr/local/share/man by default and OpenBSD does not search this location for the man pages. This can be fixed easily by adding this path of man.conf.

## Usage

dwm has very few shortcuts by default, so it is easier to remeber and get used to. The man page and suckless tutorial has a decent explanation and copying it here will not add any value. 

## Reference
1. [dwm](https://dwm.suckless.org/)
2. [tutorial](https://dwm.suckless.org/tutorial/)
