---
author: admin
comments: true
date: 2009-04-17 00:33:57+00:00
layout: post
slug: subversion-global-ignores
title: Subversion Global Ignores
wordpress\_id: 540
tags:
- mac
- svn
---

If you're a Mac OS X user running Subversion on the command line and mounting remote disks you may have run into instances where you want to globally ignore .\_\*, .AppleDouble, and \*:2e\_\* files.  To do this simply open your Terminal and edit your ~/.subversion/config file and look for the line below.

```
#global-ignores = *.o *.lo *.la #*# .*.rej *.rej .*~ *~ .#* .DS_Store
```

Uncomment it by removing the # and add the necessary additional exclusions and it will now look like this.

```
global-ignores = *.o *.lo *.la #*# .*.rej *.rej .*~ *~ .#* .DS_Store ._* .AppleDouble *:2e_*
```

Save it and you're all set.  Now you won't need to edit your svn:ignore property for every project!
