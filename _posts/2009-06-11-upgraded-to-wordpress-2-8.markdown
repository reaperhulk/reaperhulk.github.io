---
author: admin
comments: true
date: 2009-06-11 04:11:36+00:00
layout: post
slug: upgraded-to-wordpress-2-8
title: Upgraded to WordPress 2.8
wordpress\_id: 628
tags:
- fidgetr
- wordpress
---

I just upgraded this blog from 2.7.1 to 2.8 with no real issues.  I've been keeping an eye on the development of 2.8 due to Fidgetr and CDN Tools, but while Fidgetr works without any issues[^1], CDN Tools users should do the following:

1. Enable advanced options and choose "remove JS".
2. Upgrade to 2.8
3. Reload JS (if desired)

Additionally, deleting attachments does not work properly at this time due to an improvement in the wp\_delete\_attachment hook for 2.8.  I'll be releasing an update to correct both of these issues as soon as possible.

[^1]: Despite this, I've prepped a v1.3.1 with a few UI tweaks and explicit 2.8 support
