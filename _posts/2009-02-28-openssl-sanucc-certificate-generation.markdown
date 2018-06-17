---
author: admin
comments: true
date: 2009-02-28 20:45:52+00:00
layout: post
slug: openssl-sanucc-certificate-generation
title: OpenSSL SAN/UCC Certificate Generation
wordpress\_id: 321
tags:
- crypto
- openssl
- ssl
---

Signing a CSR containing subjectAltName (SAN/UCC) extensions isn't hard, but can be a daunting challenge for the OpenSSL neophyte.  We're going to use the [OpenSSL Self-Signed CA](/2009/01/18/openssl-self-signed-ca/) to accomplish this task in two ways.


### Pre-Existing SAN CSR


Either you already have a SAN CSR from another source or you generated one using the [tutorial](/2009/02/27/creating-a-subjectaltname-sanucc-csr/) from yesterday.  Inside your myca.conf file you'll need to add the following under the [ myca ] section.

```
copy_extensions	= copy
```

Now you can simply sign the CSR using the method specified in the self-signed CA post and you're all set.



### Add SAN/UCC Extensions to Existing CSR


To accomplish this add the following to your myca.conf under the [ myca\_extensions ] section.

```
subjectAltName          = @alt_names
```

Then add this section at the end of the file.

```
[alt_names]
DNS.1   = test.domain.com
DNS.2   = other.domain.com
DNS.3   = www.domain.net
```

Set the DNS entries under alt\_names to what you want (adding DNS.4 = if you need more, et cetera).  Be sure you **do not** have the copy\_extensions directive present in your conf.
Once you have done this you can sign any CSR you choose with the command specified in the self-signed CA article and it will add the specified subjectAltName attributes to the certificate.
