---
author: admin
comments: true
date: 2009-11-16 21:18:55+00:00
layout: post
slug: circumventing-hulu-regional-restrictions-in-mac-os-x
title: Bypass Hulu Regional Restrictions in Mac OS X
wordpress\_id: 780
tags:
- hulu
- linux
- mac
- vpn
---

Hulu is a great site to find new shows and catch up on old, but due to various contracts no one outside the US can use it.  This irritated some friends of mine from Canada, England, Germany, et cetera.  So I decided to write up one (very reliable) way to circumvent the Hulu geolocation checks -- using a VPN.[^1]



### Accessing Hulu Outside The US


In this case, we'll be using a small VM and the open source VPN server pptpd.  All the server side instructions below are applicable to both OS X and Windows, but the client setup is only specified for Mac OS X.



### Server Setup


First, obtain a VM from a reputable (and fast) US vendor.  The VM must be located in the US since that's our required origin.  I personally use [Slicehost](http://www.slicehost.com), but there are many others.  Once you get your login be sure you change the root password.

Install pptpd.  If you're running on Ubuntu or Debian you can simply run

```bash
apt-get install pptpd
```

Once you have pptpd installed, we'll need to add a user.  The default pptpd configuration is fine, but we'll need to edit /etc/ppp/chap-secrets. When you edit the file (using vi, nano, emacs, et cetera) you'll see this:

```bash
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
```

Client is your username, server is "pptpd", secret is your password in plaintext, and IP addresses is a range of allowed IPs.  If you're unconcerned about who might attempt to access your VPN, you can simply use a wildcard (*).  Once you've populated this file with data it will look something like this:

```bash
# Secrets for authentication using CHAP
# client	server	secret			IP addresses
testuser	pptpd	mypassword		*
```


We need to set up IPv4 forwarding, so edit /etc/sysctl.conf and uncomment the line below from the file (remove the #).

```bash
#net.ipv4.ip_forward=1
```

This will enable the behavior after a reboot, but you can enable it right now by running:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```


Now run these commands:

```bash
/sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
/sbin/iptables -A FORWARD -i eth0 -o ppp0 -m state --state RELATED,ESTABLISHED -j ACCEPT
/sbin/iptables -A FORWARD -i ppp0 -o eth0 -j ACCEPT
```

Once you've run these you can save them so they execute every time your VM boots by following [these quick instructions](http://rackerhacker.com/2009/11/16/automatically-loading-iptables-on-debianubuntu/).  This completes the server side setup.



### Client Setup


Now it's time to configure the Mac to utilize the VPN server.  Bear in mind that all traffic to the internet will be routed through your VPN server when this is active, so you'll only want to connect to your VPN when watching Hulu.[^2]

Open System Preferences and go to Network.  Click the plus sign in the lower left and choose add to add a VPN PPTP interface.  Then set the server address (the IP of your VM) and account name ("testuser" from above).
[![network_screen](/assets/media/2009/11/network_screen-300x264.png)](/assets/media/2009/11/network_screen.png)

After filling out those fields, click authentication settings and type your password, then click Okay.

[](/assets/media/2009/11/network_screen.png)[![advanced](/assets/media/2009/11/advanced-300x185.png)](/assets/media/2009/11/advanced.png)

Finally, click advanced, then click DNS and click the plus sign.  Add 4.2.2.1 as a DNS server.[^3]

[](/assets/media/2009/11/advanced.png)[![dns_fix](/assets/media/2009/11/dns_fix-300x234.png)](/assets/media/2009/11/dns_fix.png)

Save these changes and then you can click connect to test it out.  Your traffic should all be routed through the VPN and since the endpoint is located in the US Hulu should work just fine!

[^1]: There are many other ways, including just proxying Hulu traffic from the browser and Flash plugin, but I'm not going to cover those methods.

[^2]: This can be alleviated by using a split tunnel if you want to go to the trouble.

[^3]: Our PPTP server doesn't announce its own DNS by default.
