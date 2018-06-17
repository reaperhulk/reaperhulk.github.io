---
author: admin
comments: true
date: 2010-06-15 21:41:17+00:00
layout: post
slug: fixing-growlmail-in-10-6-4-mail-4-3
title: Fixing GrowlMail in 10.6.4 (Mail 4.3)
wordpress\_id: 1129
tags:
- bash
- growlmail
- mac
- mail
---

**Update 2: Fix for [10.6.7 and Mail 4.5](/2011/03/21/fixing-growlmail-in-10-6-7-mail-4-5/)**
**Update: Fix for [10.6.5 and Mail 4.4](http://langui.sh/2010/10/14/fixing-growlmail-in-10-6-5-mail-4-4/)**

Another OS X release, another broken GrowlMail bundle.  I did a post just like this for 10.6.2.  [Check it out](/2009/11/09/fixing-growlmail-letterbox-for-mail-4-2/) if you want more background on why this occurs.



### Easy Fix


Download a pre-patched GrowlMail.bundle and drop it in your ~/Library/Mail/Bundles/ directory[^1].  If you want it available to multiple users on your system, use /Library/Mail/Bundles/.

**[Download patched bundle](/assets/media/2010/06/GrowlMail.mailbundle.zip)**

If you use this method you're all set; no need to use the command line solution below.



### Add New UUIDs to SupportedPluginCompatibilityUUIDs


If you have already had your plugins disabled by opening Mail.app you'll need to look in ~/Library/Mail (or /Library/Mail if you installed globally) and move the files back to the active bundles directory.  They'll typically be in Bundles (Disabled), so quit Mail, find them, and move them back into the proper directory.

If you have a local installation:

```bash
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "E71BD599-351A-42C5-9B63-EA5C47F7CE8E"
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "B842F7D0-4D81-4DDF-A672-129CA5B32D57"
```

Global installation:

```bash
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "E71BD599-351A-42C5-9B63-EA5C47F7CE8E"
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "B842F7D0-4D81-4DDF-A672-129CA5B32D57"
```



[^1]: ~ means your home directory if you're unfamiliar with the syntax. You can click the home icon on your Finder sidebar if you're still confused
