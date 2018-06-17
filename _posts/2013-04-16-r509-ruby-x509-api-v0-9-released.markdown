---
author: admin
comments: true
date: 2013-04-16 17:14:30+00:00
layout: post
slug: r509-ruby-x509-api-v0-9-released
title: r509 (Ruby Certificate Authority API) v0.9 Released
wordpress\_id: 1878
---

[r509](http://r509.org) v0.9 is out! This version contains huge new features, bug fixes, and quite a bit of refactoring of the API in preparation for 1.0.



### What is r509?


r509 is a Ruby gem built using OpenSSL that is designed to ease management of a public key infrastructure. The r509 API facilitates easy creation of CSRs, signing (and parsing!) of certificates, revocation (CRL/OCSP), and much more. Together with projects like r509-ocsp-responder and r509-ca-http it is intended to be a complete (RFC 3280/5280) certificate authority for use in production environments.



### Feature Highlights


A very incomplete list of the extensive changes made from 0.8.1 to 0.9.




  * Feature: Elliptic curve (ECDSA) support! You can now generate ECDSA keys and sign certificates within the normal r509 framework. This support depends on your system having EC support enabled in OpenSSL. Red Hat distributions (Fedora, RHEL, CentOS, etc) have this disabled by default. For more details check out the [CSR+key generation](http://r509.org/doc/R509/CSR.html#constructor_details) and [private key generation](http://r509.org/doc/R509/PrivateKey.html#constructor_details) docs.


  * Feature: Rewrote all extension handling to directly parse the ASN.1 structures.


  * Feature: Added support in the signer and config files for more complex certificate policy data, as well as inhibit any policy, policy constraints, and name constraints extensions.


  * Feature: Support for multiple general name types in extensions. See [GeneralName](http://r509.org/doc/R509/ASN1/GeneralName.html).


  * Feature: Automatically generated getter/setter methods added for all registered OIDs on [R509::Subject](http://r509.org/doc/R509/Subject.html). (See docs for details)


  * Bug Fix: The r509 command line script now properly self-signs with the specified message digest in interactive mode rather than always using SHA1.


  * Bug Fix: The r509 command line scripts now load using /usr/bin/env to work properly in rvm-based environments.


  * Refactor: The R509::CRL module has been refactored. [R509::CRL::Administrator](http://r509.org/doc/R509/CRL/Administrator.html) has changed significantly and R509::Crl::Parser has been renamed to [R509::CRL::SignedList](http://r509.org/doc/R509/CRL/SignedList.html) and gained new methods.


  * Refactor: Class names have been altered to be fully capitalized if they are an acronym.


  * Support Change: Due to limitations in the elliptic curve bindings Ruby 1.8.7 is no longer a supported r509 platform. Please upgrade to 1.9.3+!





#### Installing/Contributing


Just gem install r509 (and any of the other r509 gems you use) and you're ready to go! sha1sums for each gem are below if you wish to verify you have an unaltered gem.

If you'd like to contribute, just click the gem name to view the source on GitHub! Contribution and feedback is always welcome.



| Gem Name (with GitHub Link) | Version | sha1sum |
| --------------------------- |:-------:| -------:|
| [r509](https://github.com/r509/r509) | 0.9 | 84606ed495d7cf70d7b968c9bd3487f2cb236d49 |
| [r509-ocsp-responder](https://github.com/r509/r509-ocsp-responder) | 0.3.2 | 8724fdcfbb7ddb54c9b9b3cd833ddb60f63e627b |
| [r509-ca-http](https://github.com/r509/r509-ca-http) | 0.2 | a3f1095e529d1cc387934c289f3e4a9778e3d606 |
| [r509-middleware-validity](https://github.com/r509/r509-middleware-validity) | 0.2.1 | 46025f7e4e9f2ff13c735462764a14751adc9812 |
| [r509-middleware-certwriter](https://github.com/r509/r509-middleware-certwriter) | 0.2.1 | 89d4ca24c3e0191a9c696cc4b367bbf3805128d4 |
| [r509-validity-redis](https://github.com/r509/r509-validity-redis) | 0.4.1 | 3d8c168a8d12efe706c4cc1d0aa34e34357bf5e8 |


#### Ruby CA Tutorial


I've updated my how-to on [building a CA with r509](https://langui.sh/2012/11/02/building-a-ca-r509-howto/) to support the new version as well, so if you're just getting started with r509 check it out!



#### Future Release Plans


r509 is largely feature complete. The intent is to release 1.0 in a few months when I'm comfortable with long-term support for the existing API. Releases will conform to the principles of [semantic versioning](http://semver.org).
