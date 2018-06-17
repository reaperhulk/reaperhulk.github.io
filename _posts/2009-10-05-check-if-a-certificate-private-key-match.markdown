---
author: admin
comments: true
date: 2009-10-05 14:52:30+00:00
layout: post
slug: check-if-a-certificate-private-key-match
title: Check If A Certificate & Private Key Match
wordpress\_id: 678
tags:
- crypto
- openssl
- x509
---

Check if an SSL certificate and private key match in two simple commands.  The OpenSSL commands below will require you to replace <file> with your file's name.

For your SSL certificate:[^1]

```bash
openssl x509 -noout -modulus -in  | md5sum
```

For your RSA private key:

```bash
openssl rsa -noout -modulus -in  | md5sum
```


The output of these commands should be identical.  If it isn't, your keys do not match.

[^1]: The pipe to md5sum is solely to make the output shorter and easier to visually compare
