---
layout: post
title: PyCA cryptography 2.0
---

[pyca/cryptography](https://github.com/pyca/cryptography) 2.0 was been released today! This sounds like a big release, but it's really just the 20th feature release we've done since starting the project[^1].

But if it's just a typical feature release then why the blog post? Well, coincidentally 2.0 actually is a huge release! In addition to adding support for ChaCha20Poly1305, AES-CCM, X25519 key exchange, Diffie-Hellman improvements, and various X.509 additions, we're also shipping a [manylinux1](https://www.python.org/dev/peps/pep-0513/) wheel. This means users on manylinux1 compatible distributions (basically any Linux using glibc and running on x86 or x86\_64) will no longer need a compiler (or even OpenSSL) when installing cryptography. Just `pip install cryptography` and you're ready to go without worrying about whether or not your OpenSSL is recent enough to support ChaCha20Poly1305 or if X25519 will be available.

Of course, nothing is perfect and this magical new world of Linux binaries comes with a few caveats:

* You need to be on pip 8.1+ to get the wheel. Otherwise pip doesn't know about manylinux1 and will continue to download the sdist and compile.
* Your distribution has to use glibc. Sorry Alpine Linux users.
* Your system must be x86 or x86-64 based. Raspberry Pis or other ARM systems can't use these wheels[^2].
* Since we're bundling OpenSSL with the wheel, merely updating OpenSSL on your system is insufficient! This means you will need to upgrade cryptography whenever a new OpenSSL release occurs. Be sure to let your ops people know about this! If this is not something you can do at this time then we have documentation explaining [how to avoid the prebuilt wheel](https://cryptography.io/en/latest/installation/#building).

Finally, this change has an interesting cascading effect on pyOpenSSL. OpenSSL sets default values at compile-time for the path to trusted root certificates. This path is used when a user calls pyOpenSSL's `Context.set_default_verify_paths()` and then subsequent TLS connections using that context are verified with those roots. With a manylinux1 wheel the default path does not match the system's path. This causes validation failures on TLS connections since it is unable to find the system root store. To avoid this we released pyOpenSSL 17.1.0 last month, so be sure to upgrade pyOpenSSL when you upgrade cryptography!

[^1]: Maybe we should switch to [calver](http://calver.org), but that's a debate for another time.
[^2]: If you want to help define a specification for ARM wheels you might want to join the [wheel builders](https://mail.python.org/mailman/listinfo/wheel-builders) mailing list.
