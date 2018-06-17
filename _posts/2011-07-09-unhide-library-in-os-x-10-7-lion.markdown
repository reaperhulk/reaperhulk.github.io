---
author: admin
comments: true
date: 2011-07-09 21:32:35+00:00
layout: post
slug: unhide-library-in-os-x-10-7-lion
title: Unhide ~/Library in OS X 10.7 Lion
wordpress\_id: 1507
---

Lion hides the user home directory by default. You can navigate to it with cmd-shift-G easily, but if you happen to want it to be visible all the time just run this command in a Terminal.app window.


```bash
chflags nohidden ~/Library
```


Thanks to [Matt Pennig](http://twitter.com/#!/pennig) for the command.
