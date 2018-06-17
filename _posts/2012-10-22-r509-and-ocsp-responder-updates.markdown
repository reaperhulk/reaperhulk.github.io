---
author: admin
comments: true
date: 2012-10-22 18:36:18+00:00
layout: post
slug: r509-and-ocsp-responder-updates
title: r509 and OCSP Responder Updates
wordpress\_id: 1748
tags:
- ca
- ocsp
- r509
- ruby
- ssl
---

I just tagged v0.8 of r509 and v0.3 of r509-ocsp-responder. Here's what's new!



#### [r509](https://github.com/r509/r509)


  * Refactor R509::Validity to support a new #is\_available? method

  * Serial randomization improvements in CertificateAuthority::Signer

  * Better documentation

  * Added ::load\_from\_file static method to more objects

  * r509-parse binary for quick parsing (alpha, will be improved)


#### [r509-ocsp-responder](https://github.com/r509/r509-ocsp-responder)


  * Updated the /status endpoint to make it simpler to replace the redis backend with an alternate R509::Validity implementation

  * Print config on startup

  * Support reloading and printing of config to logs via kill -USR2 signal

  * Statistics collection now supported via [r509-ocsp-stats](https://github.com/r509/r509-ocsp-stats)

  * New code path for ~100x faster config lookups (this will not be used until at least Ruby 2.0. I have a pending patch that needs to land in trunk and eventually get released)

  * Changed default log levels

  * Switch to using hiredis for redis lookups (if you're using the redis backend)



[r509-validity-redis](https://github.com/r509/r509-validity-redis) was also updated to work with the latest OCSP responder release if you're using the redis backend.
