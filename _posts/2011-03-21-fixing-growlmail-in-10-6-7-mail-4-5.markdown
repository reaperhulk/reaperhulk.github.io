---
author: admin
comments: true
date: 2011-03-21 21:19:41+00:00
layout: post
slug: fixing-growlmail-in-10-6-7-mail-4-5
title: Fixing GrowlMail in 10.6.7 (Mail 4.5)
wordpress\_id: 1466
tags:
- growlmail
- mac
---

**Update: OS X Lion (10.7) users. Mail bundles have changed significantly in Mail 5.0 so the quick fix hack will not work to get GrowlMail functioning again. There will need to be some actual development work to make it compatible. Sorry!**

Welcome back! We've been here before haven't we?  [Check out](/2009/11/09/fixing-growlmail-letterbox-for-mail-4-2/) my previous post on this issue if you want more background on why this occurs.



### Easy Fix


Download a pre-patched GrowlMail.bundle and drop it in your ~/Library/Mail/Bundles/ directory[^1].  If you want it available to multiple users on your system, use /Library/Mail/Bundles/.

**[Download GrowlMail 10.6.7 mailbundle](/assets/media/2011/03/GrowlMail.mailbundle.zip)**

If you use this method you're all set; no need to use the command line solution below.



### Add New UUIDs to SupportedPluginCompatibilityUUIDs


If you have already had your plugins disabled by opening Mail.app you'll need to look in ~/Library/Mail (or /Library/Mail if you installed globally) and move the files back to the active bundles directory.  They'll typically be in Bundles (Disabled) or something similar, so quit Mail, find them, and move them back into the proper directory.

If you have a local installation:

```bash
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "1C58722D-AFBD-464E-81BB-0E05C108BE06"
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "9049EF7D-5873-4F54-A447-51D722009310"
```

Global installation:

```bash
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "1C58722D-AFBD-464E-81BB-0E05C108BE06"
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "9049EF7D-5873-4F54-A447-51D722009310"
```



[^1]: ~ means your home directory if you're unfamiliar with the syntax. You can click the home icon on your Finder sidebar if you're still confused
