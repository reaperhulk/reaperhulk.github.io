---
author: admin
comments: true
date: 2009-03-28 20:28:28+00:00
layout: post
slug: using-openssl-s_time
title: Using OpenSSL s_time
wordpress\_id: 431
tags:
- benchmark
- crypto
- openssl
- ssl
---

Recently I needed to do some performance testing of an SSL instance on a VM.  I considered using JMeter, but decided to use OpenSSL to get a rudimentary picture instead.

To obtain a basic result, we connect to the server and pull the /index.php file.  You can specify whatever file you'd like to download, or none at all if you simply want to test connections.[^1]

```bash
openssl s_time -www /index.php -new -connect www.trustwave.com:443
```


Your result will look something like this:

```
No CIPHER specified
Collecting connection statistics for 30 seconds
ttttttttttttttttttttttttttttttttttttttttttttttttttttttttt
159 connections in 5.82s; 27.32 connections/user sec, bytes read 62328
159 connections in 31 real seconds, 392 bytes read per connection
```

If you'd like to get more specific with performance testing you can even use the -ciphers parameter to explicitly choose the negotiated cipher.  You can obtain a list of available ciphers with "openssl ciphers".

[^1]: If you would prefer to reuse connections rather than create a new one for each request replace -new with -reuse.
