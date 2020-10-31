---
layout: post
date: 2017-05-11
title: Setting Up Failover Interface Using Trunk
tags: ["openbsd"]
---

Often times you want the convenience of seemlessly switching over from your
wired connection to wireless connection and vice versa. This is where the
trunk interface comes into picture.

## What is trunk
trunk is a pseudo interface that allows aggregation of multiple network
interfaces as one virtual trunk interface(man 4 trunk). Though trunk supports
multiple protocols our focus here is on setting it up in failover mode.

In the failover mode, the first interface added will be a master interface
and any subsequent interfaces added will be failover interfaces. In this
example, I will set up my ethernet interface em0 as master and wireless
interface urtwn0 as a failover interface by adding them to trunk0. What
this means is whenever I disconnect my ethernet cable from the laptop,
it seamlessly switches to wireless.

You will not see an ip address assigned to member interfaces themselves
but ip is added to the trunk0 interface.

## How to set up trunk
OpenBSD man page for trunk(4) has an example to set it up using command
line options. This is good if you just need once or you just want to test
it. But you will have to run the commands after every reboot. If you like
to make this permanent you can follow below steps. 


The below commands assume that your wired interface is em0 and wireless
interface is urtwn0.

Bring up the wired interface
```
	# echo up > /etc/hostname.em0
```

Bring up the wireless interface. You need to replace _mynwid_ with your 
SSID and replace _mypasswd_ with your wifi password.
```
	# echo nwid mynwid > /etc/hostname.urtwn0
	# echo wpakey mypasswd >> /etc/hostname.urtwn0
	# echo up >> /etc/hostname.urtwn0
```

Now we will set up the trunk in a failover mode and add these two interfaces.
```
	# cat > /etc/hostname.trunk0 <<EOF
	trunkproto failover
	trunkport em0
	trunkport urtwn0
	dhcp
	rtsol
	EOF
```
The dhcp in file indicates to get the ip address dynamically and rtsol will
get the ipv6 address.

## Reference
1. [trunk(4)](http://man.openbsd.org/trunk.4)
