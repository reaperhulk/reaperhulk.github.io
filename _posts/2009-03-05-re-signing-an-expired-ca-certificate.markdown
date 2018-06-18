---
author: admin
comments: true
layout: post
slug: re-signing-an-expired-ca-certificate
title: Re-Signing An Expired CA Certificate
wordpress\_id: 330
tags:
- crypto
- openssl
- ssl
---

On rare occasions you may find yourself with a self-signed internal CA that has expired while you are still using certificates issued from the CA.  One potential solution to this problem is to self-sign a new cert with identical fields using the private key from the old certificate.[^1]

You can fill in almost all the fields using the interactive prompt, but to ensure maximum compatibility be sure every field matches exactly.  You will also need to set the serial number of the certificate via the -set\_serial parameter (openssl takes this argument in decimal form, not hex)[^2].

```bash
openssl req -new -x509 -key previousprivatekey.pem -set_serial 0000 -out newroot.cer
```

You now have a new root certificate that will work with your previously issued certificates!

[^1]: In general this is **very bad practice**, but this article presupposes that you recognize this and it is still necessary.

[^2]: If you fail to set the serial identically Microsoft OSes will chain the certificate correctly but OpenSSL will fail.
