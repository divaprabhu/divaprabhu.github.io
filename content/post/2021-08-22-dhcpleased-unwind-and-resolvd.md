---
layout: post
date: 2021-08-22
title: dhcpleased, unwind and resolvd
tags: ["dhcpleased", "resolvd", "unwind", "openbsd"]
---

dhcpleased is a new daemon in OpenBSD that will dynamically configure the client network. It monitors network interfaces for IP address proposals from DHCP, IPv6 router advertisements to configure the network and then sets the resolv.conf. unwind is a validating DNS resolver that runs on the client machine. resolvd handles the contents of /etc/resolv.conf. 

The 3 deamons effectively replace my dhclient and unbound set up and make the life much easier. For more details on the advantages of these daemons and what problem they solve, please read the man pages and reference links. Here I will give the configuration.

## Configuration

We will first create a blocklist file that dictates which domains should be blocked by unwind. Building of the file is similar to building a blocklist file in my previous post on unbound.

```
		_unbounddir="/var/unbound"
		_blocklist="blocklist"
		_tmpfile="$(mktemp)" && echo $_tmpfile

		_disconad="https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt"
		_discontrack="https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt"
		_stevenblack="https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts"
		_zeustracker="https://zeustracker.abuse.ch/blocklist.php?download=domainblocklist"

		ftp -VMo - $_zeustracker| sed -e 's/#.*$//' -e '/^[[:space:]]*$/d' >> $_tmpfile
		ftp -VMo - $_discontrack| sed -e 's/#.*$//' -e '/^[[:space:]]*$/d' >> $_tmpfile
		ftp -VMo - $_disconad   | sed -e 's/#.*$//' -e '/^[[:space:]]*$/d' >> $_tmpfile
		ftp -VMo - $_stevenblack| sed -n '/Start/,$p' | sed -e 's/#.*$//' -e '/^[[:space:]]*$/d' | awk '/^0.0.0.0/ { print $2 }' >> $_tmpfile
		(sort -fu ${_tmpfile} | sed -e '/^[[:space:]]*$/d') > ${_blocklist} && rm $_tmpfile
		echo "$(date) $(wc -l ${_blocklist}) entries added"

		doas install -o root -g wheel -m 0644 ${_blocklist} ${_unbounddir}/${_blocklist} && rm ${_blocklist}

```

Next create an unwind config file with these contents -

```
	echo 'block list "/var/unbound/blocklist" log' | $SUDO tee -a /etc/unwind.conf

```

Now we enable the unwind daemon and start it.
```
	doas rcctl enable unwind
	doas rcctl start unwind
```

In order to use dhcpleased, all you have to do is add these lines to hostname.if files -

```
	inet autoconf
	inet6 autoconf
	up
```

And now enable both resolvd and dhcpleased

```
	doas rcctl enable resolvd
	doas rcctl enable dhcpleased
	doas rcctl start resolvd
	doas rcctl start dhcpleased
```

## Reference
1. [dhpcpleased](https://undeadly.org/cgi?action=article;sid=20210722072359&utm_source=discoverbsd)
2. [unwind](https://undeadly.org/cgi?action=article;sid=20190128061321)
3. [resolvd](https://undeadly.org/cgi?action=article;sid=20210225084959)
