---
author: admin
comments: true
date: 2009-10-31 16:32:37+00:00
layout: post
slug: create-a-2048-bit-key-via-openssl
title: Create A 2048-bit Key Via OpenSSL
wordpress\_id: 741
tags:
- openssl
---

We are fast approaching the date where [NIST](http://www.nist.gov) has recommended that end entities stop utilizing 1024-bit private keys.  OpenSSL, however, currently defaults to creating 1024-bit keypairs. To create a 2048-bit private key and corresponding CSR (which you can send to a certificate authority to obtain your SSL certificate):

```bash
openssl req -new -nodes -newkey rsa:2048 -keyout mydomain.key -out mydomain.csr
```

This command will make a 2048-bit key, run the interactive prompt to populate the fields of the certificate signing request, and leave the private key unencrypted (-nodes).  You can remove -nodes if you wish, but encrypting the private key will require you to type the password every time you start an application (like apache) that uses it.
