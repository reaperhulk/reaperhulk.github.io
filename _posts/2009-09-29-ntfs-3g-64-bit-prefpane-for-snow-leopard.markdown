---
author: admin
comments: true
date: 2009-09-29 16:57:51+00:00
layout: post
slug: ntfs-3g-64-bit-prefpane-for-snow-leopard
title: NTFS-3G 64-bit PrefPane For Snow Leopard
wordpress\_id: 662
tags:
- mac
- os x
- prefpane
---

Update: A version of NTFS-3G with native 64-bit PrefPane support has been [released](http://macntfs-3g.blogspot.com/).  The instructions below have been left for archival purposes, but are no longer required.

I use NTFS-3G + MacFUSE to write to NTFS disks on occasion, but I hate restarting System Preferences just to run a 32-bit PrefPane when I want to view/change some settings.  Behold the power of open source!

To compile a 64-bit System Preferences compatible version of this item simply




  1. Grab the [source](http://downloads.sourceforge.net/catacombae/ntfs-3g_prefpane-0.9.8-src.tar.bz2?use_mirror=) for the [NTFS-3G](http://macntfs-3g.blogspot.com/) PrefPane


  2. Right click and get info on the build target to add x86\_64, turn on Objective-C garbage collection to "supported"[^1]


  3. Build the target.


...and you're all set.  You can [download](/assets/media/2009/09/NTFS-3G.prefPane.zip) my copy, but I don't recommend using it in 10.5 or 10.4.  Of course, you don't need a 64-bit PrefPane prior to 10.6 so why would you download it anyway?

[^1]: If you set it to supported it should work in OS X 10.5 still
