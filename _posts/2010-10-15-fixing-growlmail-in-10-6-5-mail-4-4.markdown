---
author: admin
comments: true
date: 2010-10-15 02:58:01+00:00
layout: post
slug: fixing-growlmail-in-10-6-5-mail-4-4
title: Fixing GrowlMail in 10.6.5 (Mail 4.4)
wordpress\_id: 1369
tags:
- growlmail
- mac
---

**Update: Fix for [10.6.7 and Mail 4.5](/2011/03/21/fixing-growlmail-in-10-6-7-mail-4-5/)**

Another OS X release (beta as of this posting), another broken GrowlMail bundle.  I did a post just like this for 10.6.2 (and 10.6.4).  [Check it out](/2009/11/09/fixing-growlmail-letterbox-for-mail-4-2/) if you want more background on why this occurs.



### Easy Fix


Download a pre-patched GrowlMail.bundle and drop it in your ~/Library/Mail/Bundles/ directory[^1].  If you want it available to multiple users on your system, use /Library/Mail/Bundles/.

**[Download GrowlMail 10.6.5 mailbundle](/assets/media/2010/10/GrowlMail-10.6.5.mailbundle.zip)**

If you use this method you're all set; no need to use the command line solution below.



### Add New UUIDs to SupportedPluginCompatibilityUUIDs


If you have already had your plugins disabled by opening Mail.app you'll need to look in ~/Library/Mail (or /Library/Mail if you installed globally) and move the files back to the active bundles directory.  They'll typically be in Bundles (Disabled), so quit Mail, find them, and move them back into the proper directory.

If you have a local installation:

```bash
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "857A142A-AB81-4D99-BECC-D1B55A86D94E"
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "BDD81F4D-6881-4A8D-94A7-E67410089EEB"
```

Global installation:

```bash
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "857A142A-AB81-4D99-BECC-D1B55A86D94E"
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "BDD81F4D-6881-4A8D-94A7-E67410089EEB"
```



[^1]: ~ means your home directory if you're unfamiliar with the syntax. You can click the home icon on your Finder sidebar if you're still confused
