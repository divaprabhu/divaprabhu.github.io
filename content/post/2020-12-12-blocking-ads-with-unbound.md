---
layout: post
date: 2020-12-12
title: Blocking ads with unbound
tags: ["unbound", "openbsd"]
---

So you have set up a WireGuard VPN on the Raspbeery Pi and would like to block ads and trackers for all the connected clients on the network. This last part of the series will explain how to do it using unbound(8).

## Configuring unbound
Unbound is already available by default in OpenBSD. By default it is chrooted to /var/unbound. The configuration file is located at /var/unbound/etc/unbound.conf. Here is what I have in unbound.conf. In the access control, I have included the vpn network 10.0.0.0/24 so that unbound can serve DNS on this network. 

The include statement will include blocklist file that contains all the domains we would like to block. We will build such a file in the next section.
```
server:
	include: /var/unbound/etc/blocklist.conf
	verbosity: 1

	interface-automatic: yes

	access-control: 192.168.0.0/24 allow
	access-control: 10.0.0.0/24 allow
	access-control: 127.0.0.0/8 allow
	access-control: ::1 allow

	log-servfail: yes

	hide-identity: yes
	hide-version: yes

	aggressive-nsec: yes

	auto-trust-anchor-file: /var/unbound/db/root.key
	val-log-level: 2

remote-control:
	control-enable: yes
	control-interface: /var/run/unbound.sock
```

We will now download the trust anchor file using the unbound-anchor. The trust-anchor is used for DNSSEC validation

```
	doas unbound-anchor -a "/var/unbound/db/root.key"
```

## Building blocklist file

We will use the files from [blocklistproject](https://blocklistproject.github.io) for building a blocklist file. Using all the files from the list will consume almost 1GB of ram so you can pick and chose which one you want to use. An alternative is to use other popular lists like [StevenBlack](https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts). [Tumfatig](https://www.tumfatig.net/wp-content/uploads/2019/04/unbound-adhosts.txt) has a nice script to automatically parse multiple lists and build a blocklist file for unbound.

```
	_tmpfile="$(mktemp)"
	lists=""
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/abuse-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/ads-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/crypto-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/drugs-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/facebook-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/fraud-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/gambling-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/malware-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/phishing-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/piracy-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/porn-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/ransomware-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/redirect-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/scam-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/tiktok-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/tracking-nl.txt"
	lists="$lists https://blocklistproject.github.io/Lists/alt-version/youtube-nl.txt"
	for list in $lists; do
		echo "$(date) Getting $list"
		ftp -VMo - $list | sed -e 's/#.*$//' -e '/^[[:space:]]*$/d' >> ${_tmpfile}
	done

	sort -fu ${_tmpfile} | \
	sed -e '/^[[:space:]]*$/d' | \
	awk '{
	  print "local-zone: \"" $1 "\" static"
	}' > blocklist.conf && rm $_tmpfile

	echo "$(date) $(wc -l blocklist.conf) entries added"
	doas install -o root -g wheel -m 0644 blocklist.conf /var/unbound/etc/blocklist.conf && \
	rm blocklist.conf 
```

The trick is to build a series of local-zone clauses with no corresponding local-data. Using static in such cases will send out a NXDOMAIN. We are all set with configurations. The final step is to enable and start the unbound. 

```
	doas rcctl enable unbound
	doas rcctl set unbound timeout 60
	doas rcctl restart unbound
```

I had to set the timeout to 60 instead of default 30 that was resulting in failure of unbound startup.

## Reference
1. [Tumfatig](https://www.tumfatig.net/wp-content/uploads/2019/04/unbound-adhosts.tx://www.tumfatig.net/wp-content/uploads/2019/04/unbound-adhosts.txt)
2. [unbound.conf(5)](https://man.openbsd.org/unbound.conf)
