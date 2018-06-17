---
author: admin
comments: true
date: 2009-03-20 23:08:53+00:00
layout: post
slug: creating-a-pkcs7-p7b-using-openssl
title: Creating a PKCS7 (P7B) Using OpenSSL
wordpress\_id: 435
tags:
- crypto
- openssl
- ssl
---

Continuing the howto nature of this blog (and its peculiar obsession with OpenSSL), here's a primer on packaging an arbitrary number of certificates into a single PKCS7 container.  These files are quite useful for installing multiple certificates on Windows servers.  They differ from PKCS12 (PFX) files in that they can't store private keys.  If you need to [generate a PKCS12](http://langui.sh/2009/01/24/generating-a-pkcs12-pfx-via-openssl/) then head to that article instead.

This example assumes that you have 2 different certificate files, each in PEM (Base64) format.  You can add as many -certfile elements as you want to package in the file.  Additionally, concatenated certificate chains are supported.[^1]

```bash
openssl crl2pkcs7 -nocrl -certfile cert1.cer -certfile cert2.cer -out outfile.p7b
```

[^1]: If you wish to provide DER encoded input files (or have DER output) you can use the -inform DER or -outform DER directives.
