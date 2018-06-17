---
author: admin
comments: true
date: 2012-09-24 21:18:46+00:00
layout: post
slug: ruby-ocsp-responder
title: Ruby OCSP Responder
wordpress\_id: 1738
tags:
- ca
- ocsp
- r509
- ruby
- ssl
---

I've been remiss in not mentioning this project, but along with the open source ruby certificate authority project [r509](http://github.com/r509/r509) we've also built and released a full open source OCSP responder written in ruby. [r509-ocsp-responder](http://github.com/r509/r509-ocsp-responder), as we've catchily named it, is designed to conform to [RFC 2560](http://www.ietf.org/rfc/rfc2560.txt) and [5019](http://www.ietf.org/rfc/rfc5019.txt) and allow you to quickly and easily get a full OCSP responder up and running with a (relative) minimum of configuration.

r509-ocsp-responder is designed to be a "known good" responder. This means it will respond with an UNKNOWN if the responder is queried about a certificate it is unaware of even if it is configured to respond for that CA. This is in contrast to many responders, which are designed as "known bad" and will reply VALID unless the certificate is known to be revoked. We have written a redis-based validity checker ([r509-validity-redis](http://github.com/r509/r509-validity-redis)) that you can use, or you can easily write your own writer/checker backend.

If you happen to want to build an entire CA stack, we've got one last project (+ some middleware) for you. [r509-ca-http](http://github.com/r509/r509-ca-http) is a RESTful interface for issuing certificates. It also works with [r509-middleware-validity](https://github.com/r509/r509-middleware-validity) to automatically write issuance/revocation data to the OCSP responder's redis DB.

We'll be improving the documentation around these ancillary projects shortly, but feel free to dive in now (as several of you already have!).
