---
author: admin
comments: true
date: 2010-02-25 01:16:59+00:00
layout: post
slug: sni-support-in-chromium-os-x
title: SNI Support in Chromium OS X
wordpress\_id: 1111
tags:
- chromium
- sni
- ssl
---

As of [r39934](http://src.chromium.org/viewvc/chrome?view=rev&revision=39934) Chromium now supports the server\_name TLS extension (server name indication) in OS X ([latest build](http://build.chromium.org/buildbot/continuous/mac/LATEST/)).  This support requires OS X 10.5.7 or later.  Hopefully it'll make its way into a dev/beta/stable release of Google Chrome itself soon.

For those who are more curious than they ought to be about how I wrote this patch... Apple added support in their Secure Transport library for the server\_name TLS extension, but has not updated their [documentation](http://developer.apple.com/mac/library/DOCUMENTATION/Security/Reference/secureTransportRef/Reference/reference.html).  As of 10.5.7 (or possibly 10.5.6) the SSLSetPeerDomainName function -- which is ostensibly used for OS level certificate verification -- causes OS X to send the server\_name extension in the TLS client hello.  However, since Chromium doesn't use OS X's built-in verification it wasn't passing this data through prior to the patch.

To test you can hit up my IDN SNI site [https://☣.ws/](https://☣.ws/) or [https://alice.sni.velox.ch/](https://alice.sni.velox.ch/). The former will throw a certificate error if you are on a non-SNI enabled browser and the latter will have text stating that the SNI extension is missing.
