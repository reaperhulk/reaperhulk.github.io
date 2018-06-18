---
author: admin
comments: true
date: 2010-01-10 21:29:47-05:00
layout: post
slug: parsing-a-crl-with-openssl
title: Parsing A CRL With OpenSSL
wordpress\_id: 952
tags:
- crl
- openssl
---

Short and sweet.  This command will parse and give you a list of revoked serial numbers:

```bash
openssl crl -inform DER -text -noout -in mycrl.crl
```

Most CRLs are DER encoded, but you can use -inform PEM if your CRL is not binary.  If you're unsure if it is DER or PEM open it with a text editor.  If you see -----BEGIN X509 CRL----- then it's PEM and if you see strange binary-looking garbage characters it's DER.


