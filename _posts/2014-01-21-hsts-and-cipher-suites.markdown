---
layout: post
title: HSTS and Cipher Suites
---

[SSL Labs](https://www.ssllabs.com) has posted a [blog post](http://blog.ivanristic.com/2014/01/ssl-labs-stricter-security-requirements-for-2014.html) outlining the updates they've made to their tool for 2014. I have used this opportunity to fix the last few browsers that weren't getting forward secrecy when talking to this blog, as well as turning on [HSTS](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) and finally setting up a 301 redirect from http to https. For those who are curious here's the TLS configuration for my nginx vhost:

```
add_header Strict-Transport-Security max-age=31536000;
ssl                  on;
ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
ssl_stapling on;
ssl_session_cache    shared:SSL:10m;
ssl_session_timeout  10m;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:  ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-RC4-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:RC4-SHA;
ssl_prefer_server_ciphers on;
```
