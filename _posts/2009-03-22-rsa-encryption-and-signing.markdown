---
author: admin
comments: true
date: 2009-03-22 04:02:51+00:00
layout: post
slug: rsa-encryption-and-signing
title: RSA Encryption and Signing
wordpress\_id: 440
tags:
- aes
- crypto
- openssl
- rsa
- ssl
---

OpenSSL provides several tools that allow you to RSA encrypt/sign arbitrary data files.  Of course, directly RSA encrypting large volumes of data is impractical because the encrypted/signed data cannot exceed the size of the key material.  This is one of the reasons why SSL connections typically handshake and then pass an AES (or RC4, et cetera) key to do symmetric encryption thereafter.[^1]

Generate a private key. You can change the last number to the preferred modulus size.  Keys greater than 4096-bit will take a long time to generate.[^2]

```bash
openssl genrsa -out private.pem 4096
```

With the private key we can now encrypt the data.

```bash
openssl rsautl -encrypt -inkey private.pem -in publicfile -out privatefile
```

To decrypt just reverse it.

```bash
openssl rsautl -decrypt -inkey private.pem -in privatefile -out publicfile
```


If you would rather sign the data...

```bash
openssl rsautl -sign -inkey private.pem -in filetosign -out signed_data
```

To verify the signature just use -verify.[^3]

```bash
openssl rsautl -verify -inkey private.pem -in signed_data
```

[^1]: Another big reason is speed.  AES is much, much faster than RSA.

[^2]: If you attempt to encrypt or sign data larger than your key length allows, you will receive an error similar to this: 23465:error:0406D06E:rsa routines:RSA\_padding\_add\_PKCS1\_type\_2:data too large for key size:rsa\_pk1.c:151:

[^3]: You can also use -hexdump or -raw to view the data in those forms.
