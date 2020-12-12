---
layout: post
date: 2020-12-04
title: Setting up WireGuard on OpenBSD
tags: ["wireguard", "openbsd"]
---

In the previous post we installed OpenBSD on Raspberry Pi 4. If you have not already done, it is time to run syspatch and apply patches to the system. 

In this first part we will configure WireGuard VPN endpoint with a road warrior setup. As of 6.8, WireGuard is available natively on OpenBSD. We will only use tools available on the default install of OpenBSD.

## Setting up the server
WireGuard works by exchanging public keys. Let us first create keypair for the server. We will use openssl rand command to generate a private key. We then create a wg(4) interface and use it to extract corresponding public key. I have used the udp port 443 for incoming connections. We will open this port later when configuring pf. The VPN network is in subnet 10.0.0.0/24 and the server has an ip of 10.0.0.1.

```
	mkdir -p ~/wireguard/server
	cd wireguard
	openssl rand -base64 32 > server/private.key
	doas ifconfig wg0 destroy
	doas ifconfig wg0 create wgkey "$(cat server/private.key)" wgport 443
	doas ifconfig wg0 | grep wgpubkey | cut -d " " -f 2 > server/public.key
	doas ifconfig wg0 10.0.0.1
```

We will also create a hostname.if(5) file so that the WireGuard interface comes up automatically.
```
	echo "wgkey $(cat server/private.key) wgport 443" > server/hostname.wg0
	echo "inet 10.0.0.1 255.255.255.0" >> server/hostname.wg0
	doas cp server/hostname.wg0 /etc/hostname.wg0
```

Since the server needs to forward packets between WireGuard and egress interface, we need to enable ip forwarding.

```
	doas sysctl net.inet.ip.forwarding=1
	echo "net.inet.ip.forwarding=1" | doas tee -a /etc/sysctl.conf
```

We will also update the pf rules to open up port 443 for WireGuard VPN traffic and nat all traffic from WireGuard to egress interface.

```
	echo "" | doas tee -a /etc/pf.conf
	echo "pass in on egress proto udp from any to any port $WGPORT" | doas tee -a /etc/pf.conf
	echo "pass on wg0" | doas tee -a /etc/pf.conf
	echo "pass out on egress inet from (wg0:network) nat-to (egress:0)" | doas tee -a /etc/pf.conf
	doas pfctl -vvf /etc/pf.conf
```

This ends the server configuration. Next we will configure a client.

## Setting up a client

We will generate client keys and config for iPhone. I will also build a configuration that you can use on an OpenBSD client. Similar to what we did for server, we will first generate private and public key. We create a new temporary interface wg1 since wg0 is already taken.

```
	mkdir iPhone
	openssl rand -base64 32 > openbsd/private.key
	doas ifconfig wg1 destroy
	doas ifconfig wg1 create wgkey "$(cat iPhone/private.key)"
	doas ifconfig wg1 | grep wgpubkey | cut -d " " -f 2 > iPhone/public.key
	doas ifconfig wg1 destroy
```
Next we create a config file that can be used with smartphones or linux. We also update the server hostname.if(5) file to add an entry of new peer. As seen from configuration, the iPhone is assigned an ip address of 10.0.0.2.

```
	echo "" >> server/hostname.wg0
	echo "# [iPhone]" >> server/hostname.wg0
	echo "wgpeer $(cat server/public.key) wgaip 10.0.0.2" >> server/hostname.wg0
	doas ifconfig wg0 wgpeer $(cat iphone/public.key) wgaip 10.0.0.2
	doas cp server/hostname.wg0 /etc/hostname.wg0

	echo "[Interface]" > iPhone/wg.conf
	echo "PrivateKey = $(cat iPhone/private.key)" >> iPhone/wg.conf
	echo "Address = 10.0.0.2" >> iPhone/wg.conf
	echo "DNS = 10.0.0.1 1.1.1.1" >> iPhone/wg.conf
	echo "" >> iPhone/wg.conf
	echo "[Peer]" >> iPhone/wg.conf
	echo "PublicKey = $(cat server/public.key)" >> iPhone/wg.conf
	echo "AllowedIPs = 0.0.0.0/0" >> iPhone/wg.conf
	echo "Endpoint = my.domain.name:443" >> iPhone/wg.conf
```

I have used VPN server as a DNS server but actually there is no DNS server running on it. In this next part, we will actually set up unbound on it so that it can serve DNS. Also end point is set to my.domain.name which you should replace it with dns name of your server or your server's static ip. If you don't have a domain, you can use services like dyndns or freemyip.

You can tranfer this config file to your iPhone and open it with WireGuard app to set up the VPN. I usually start a httpd server and copy the config to htdocs until it can be downloaded to your phone. Create from file or archive option from the WireGuard app can also import the config file. Other alternative is to install libqrencode and generate the qr code which can then be scanned by WireGuard app.

```
	cat iPhone/wg.conf | qrencode -t ansiutf8
```

To connect an OpenBSD client, we instead generate a hostname.if(5) file and then modify the routing table to pass all traffic via wg(4) interface. As before let us generate the keypair and set the ip of this client to 10.0.0.3.

```
	mkdir OpenBSD
	openssl rand -base64 32 > OpenBSD/private.key
	doas ifconfig wg1 destroy
	doas ifconfig wg1 create wgkey "$(cat OpenBSD/private.key)"
	doas ifconfig wg1 | grep wgpubkey | cut -d " " -f 2 > OpenBSD/public.key
	doas ifconfig wg1 destroy

	echo "" >> server/hostname.wg0
	echo "# [OpenBSD]" >> server/hostname.wg0
	echo "wgpeer $(cat server/public.key) wgaip 10.0.0.3" >> server/hostname.wg0
	doas ifconfig wg0 wgpeer $(cat OpenBSD/public.key) wgaip 10.0.0.3
	doas cp server/hostname.wg0 /etc/hostname.wg0

```

Next we build a hostname.if(5) file. At the beginning there is a 5s sleep. This is to make sure my wireless connection in laptop is up and running and can resolve the end point during system reboot. The gateway ip is your current networks gateway, if your on home network behind a router you can put your router's ip here.

```
		echo '! echo "Connecting to VPN..." && sleep 5' > OpenBSD/hostname.wg0
		echo 'up' >> OpenBSD/hostname.wg0
		echo "wgkey $(cat OpenBSD/private.key)" >> OpenBSD/hostname.wg0
		echo "wgpeer $(cat server/public.key) wgendpoint my.domain.name 443 wgaip 0.0.0.0/0" >> OpenBSD/hostname.wg0
		echo "inet 10.0.0.3" >> OpenBSD/hostname.wg0
		echo "! route add -priority 2 my.domain.name 192.168.0.1" >> OpenBSD/hostname.wg0
		echo "! route add -priority 7 default 10.0.0.1" >> OpenBSD/hostname.wg0
```
Copy this file to your laptop and run "doas sh /etc/netstart" and you should see your machine connected to VPN.

## Reference
1. [WireGuard man page] (https://man.openbsd.org/wg)
2. [xocs] (https://xosc.org/wireguard.html)
