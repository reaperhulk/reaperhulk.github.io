---
author: admin
comments: true
layout: post
slug: localizing-safari-5-extensions
title: Localizing Safari 5 Extensions
wordpress\_id: 1302
tags:
- extension
- safari
---

**Update:** If you're a registered developer you can check [this thread](https://devforums.apple.com/thread/53451?tstart=0) for more information. The gist is that settings strings are not localizable at this time.  I've filed a bug 8105949 against it (I recommend everyone report one as well if they want this fixed).

I've gotten a great deal of requests to localize my Safari extensions into various languages.  Unfortunately, Apple hasn't released any documentation on whether or not the Info.plist data can be localized.

In normal Mac OS X applications you can define key/value pairs using a _Language.lproj_ directory with an _InfoPlist.strings_ file.  OS X then automatically uses the translated strings based on the primary language.  As this seemed a likely candidate for i18n support, I've been experimenting with this for the past day or two...without success.  So now I'm pleading with you, the internet! Is there a way to do i18n in extensions?  Or will we need to wait for Safari 5.x to add this (critical) feature?
