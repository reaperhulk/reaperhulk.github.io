---
layout: post
title: Cryptography 0.4 Released!
tags:
- python
---
Update: Corrected to note that the release is 0.4, not 0.5. Whoops!

[cryptography](https://cryptography.io) v0.4 has been released to [PyPI](https://pypi.python.org/pypi/cryptography). New in this version:

* SEED symmetric cipher support
* RSA encryption and decryption
* DSA signing and verification
* CMAC support
* Significantly expanded OpenSSL bindings (including NPN support) that PyOpenSSL and other downstream projects can consume.
* Windows 64-bit Python wheels

We'll be spending our next release cycle working on asymmetric key serialization, improved support for additional backends, opaque keys, and undoubtedly other random new features. [Star us on GitHub](https://github.com/pyca/cryptography) if you want to see what we're up to.
