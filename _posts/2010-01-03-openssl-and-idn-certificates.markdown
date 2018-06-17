---
author: admin
comments: true
date: 2010-01-03 17:34:46+00:00
layout: post
slug: openssl-and-idn-certificates
title: OpenSSL and IDN Certificates
wordpress\_id: 946
tags:
- idn
- openssl
---

As internationalized domain names (IDN) proliferate more people need to test with, and ultimately purchase, IDN certificates.  If you need to generate a CSR or even a self-signed certificate for an internationalized domain follow these steps:




  1. Take the UTF-8 characters and paste them into a [punycode converter](http://idnaconv.phlymail.de/) (also known as ASCII compatible encoding, or ACE).


  2. The resulting converted text will be a fairly long string that starts with "xn--".  Copy the entire thing.


  3. Now run this command.


For CSR generation[^1]:

```bash
openssl req -new -nodes -out mycsr.csr -keyout mykey.pem -newkey rsa:2048
```

For self-signed certificate generation[^2]:

```bash
openssl req -new -nodes -x509 -days 3650 -out mycert.cer -keyout mykey.pem -newkey rsa:2048
```

Either way, follow the prompts and when you reach Common Name paste the text you copied from the punycode converter.  Now you can submit your CSR to a certification authority or install the self-signed certificate for testing.

[^1]: We are generating a [2048-bit CSR](/2009/10/31/create-a-2048-bit-key-via-openssl/)

[^2]: This will generate a 10 year self-signed certificate.
