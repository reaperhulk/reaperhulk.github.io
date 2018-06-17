---
author: admin
comments: true
date: 2010-12-08 01:45:10+00:00
layout: post
slug: list-all-vhosts-currently-enabled-in-apache
title: List All VHosts Currently Enabled In Apache
wordpress\_id: 1457
tags:
- apache
---

If you want to get a list of all currently enabled virtualhosts in Apache, just type the following as root[^1]:

```bash
httpd -S
```

You'll get output that looks like this:

```
port 80 namevhost somedomain.com (/etc/httpd/conf.d/somedomain.conf:1)
port 80 namevhost another.com (/etc/httpd/conf.d/another.conf:1)
port 80 namevhost test.com (/etc/httpd/conf.d/test.conf:1)
port 80 namevhost testing.com (/etc/httpd/conf.d/testing.conf:1)
port 80 namevhost fqdn.com (/etc/httpd/conf.d/fqdn.conf:1)
```

Each line tells you port, type, the domain, and the conf file (and line) it's defined in.  A very simple command, but one I forget all the time.

[^1]: substitute apache2ctl for httpd if you're in Ubuntu/Debian
