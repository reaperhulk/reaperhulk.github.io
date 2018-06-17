---
author: admin
comments: true
date: 2009-11-03 21:01:54+00:00
layout: post
slug: ssl-vhosting-on-the-same-ip-aka-sni
title: SSL VHosting On The Same IP (aka SNI)
wordpress\_id: 747
tags:
- apache
- linux
- sni
- ssl
---

Server Name Indication (SNI), an extension to TLS, allows browsers that support it to connect to SSL hosts that do not have dedicated IPs (much like standard http virtual hosting has worked for years).  This extension, however, must be supported on both the server and client side.  Microsoft has not yet chosen to support it (maybe IIS 8?), but the Apache project did with the 2.2.12 release.  Recently, Ubuntu 9.10 Server became the first server distribution to ship with Apache and OpenSSL built with the appropriate flags, so if you'd like to follow along you can use a 9.10 VM.

In the ideal case everything is the same as a regular vhost, but you'll first need to enable SSL.  On Ubuntu this requires you to run **a2enmod** and type "ssl".  After that you'll need to add

```apache
NameVirtualHost *:443
```

to the root conf, then make your VirtualHost much like a normal one.  A very basic pair of vhosts is seen below.

```apache


	ServerAdmin webmaster@localhost

	DocumentRoot /my/doc/root
	ServerName mydomain.com
	SSLEngine On
	SSLCertificateFile /path/to/domain.crt
	SSLCertificateKeyFile /path/to/domain.key


	ServerAdmin webmaster@localhost

	DocumentRoot /my/doc/root
	ServerName mydomain2.com
	SSLEngine On
	SSLCertificateFile /path/to/domain2.crt
	SSLCertificateKeyFile /path/to/domain2.key


```

These vhosts should be placed in different includes ideally, but it isn't required.  If you just want to test with a self-signed certificate you can create one with

```bash
openssl req -new -nodes -keyout mykey.key -out mycert.cer -days 3650 -x509
```

You'll need to specify the domain name you want in the "Common Name" section.

Once you've got all this done you can restart apache and test it out!  If you test on a browser that doesn't support SNI (IE on XP) you'll get the SSL cert for the first vhost apache parses.  To disable accessing it on non-SNI hosts you can add

```apache
SSLStrictSNIVHostCheck on
```

to the root conf.  This will cause a 403 error for those browsers.

If you'd like to see an example implementation of SNI you can check out my IDN domains [https://☢.ws/](http://xn--j4h.ws) and [https://☣.ws/](https://xn--k4h.ws/).  These sites are hosted on the same IP with different SSL certificates.  I have strict host checking turned on so visiting them with a non-SNI capable browser will result in a 403 error.[^1]

[^1]: See the Wikipedia article about [Server Name Indication](http://en.wikipedia.org/wiki/Server_Name_Indication) for more information on supported browsers.
