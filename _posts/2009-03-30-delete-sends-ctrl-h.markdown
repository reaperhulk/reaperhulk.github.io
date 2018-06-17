---
author: admin
comments: true
date: 2009-03-30 05:01:12+00:00
layout: post
slug: delete-sends-ctrl-h
title: Delete Sends Ctrl-H
wordpress\_id: 463
tags:
- mac
- terminal
---

[![Delete Sends Ctrl-H in Terminal](/assets/media/2009/03/picture-1-300x239.png)](/assets/media/2009/03/picture-1.png)If you're a Mac user who utilizes Terminal.app with any regularity you have probably run into some Linux servers where the Mac Delete key behaves as forward delete instead of backspace (Ubuntu, Debian, and a few other distributions have this issue).  This is a really obnoxious problem, but fortunately there is an easy global fix.

To repair the problem you'll need to go to the Terminal.app preferences, select settings, then under the default theme you're using click the advanced tab.  Now you can check the "Delete Sends Ctrl-H" option and close the prefs.  Any existing windows will retain the old behavior but new tabs/windows will now behave as expected.
