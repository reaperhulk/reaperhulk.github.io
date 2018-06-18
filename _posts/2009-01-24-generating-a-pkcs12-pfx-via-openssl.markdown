---
author: admin
comments: true
layout: post
slug: generating-a-pkcs12-pfx-via-openssl
title: Generating a PKCS12 (PFX) Via OpenSSL
wordpress\_id: 136
tags:
- openssl
- ssl
---

Sometimes there are cases when you have a separate private key/certificate pair (perhaps with an intermediate or two) that need to be combined into a single file.  This merge can be performed on the command line using OpenSSL.

```bash
openssl pkcs12 -export -in my.cer -inkey my.key -out mycert.pfx
```

This is the most basic use case and assumes that we have no intermediates, the private key has no password associated, my.cer is a PEM encoded file, and that we wish to supply a password interactively to protect the output file.  Great, but what if that's not true?


### Common Optional Flags


**-passin** If your private key has a password, you can supply it via this flag (Example: -passin pass:mypass).  **Note:** This flag is not necessary as OpenSSL will ask you for the password interactively if it detects that the private key is passworded, but can be useful for automation.

**-in** You can add extra certificates via additional -in parameters. (Example: -in anothercert.cer)

**-inform** If your certificates are DER (binary) encoded rather than PEM (base64) use this flag (Example: -inform DER)

**-password** You can use this flag to specify the output file's password in a non-interactive fashion (Example: -password pass:mypass).  **Note:** Again, this is useful primarily to reduce interactivity and increase automation/scripting capability.

Much more advanced behavior is available, but if you need that it's probably time to check the [man page](http://www.openssl.org/docs/apps/pkcs12.html#).
