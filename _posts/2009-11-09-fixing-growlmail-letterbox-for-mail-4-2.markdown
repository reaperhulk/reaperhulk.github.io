---
author: admin
comments: true
date: 2009-11-09 22:50:00+00:00
layout: post
slug: fixing-growlmail-letterbox-for-mail-4-2
title: 'Fixing GrowlMail for Mail 4.2 '
wordpress\_id: 811
tags:
- bash
- mac
- mail
---

**Update 3: Fix for [10.6.7 and Mail 4.5](/2011/03/21/fixing-growlmail-in-10-6-7-mail-4-5/)**
**Update 2: Fix for [10.6.5 and Mail 4.4](http://langui.sh/2010/10/14/fixing-growlmail-in-10-6-5-mail-4-4/)**
**Update: Fix for [10.6.4 and Mail 4.3](http://langui.sh/2010/06/15/fixing-growlmail-in-10-6-4-mail-4-3/)**

Lately Apple has been revving the version number (and plugin compatibility UUID) of Mail.app with every version of 10.6.  This breaks bundles like GrowlMail even when they are still compatible.  The easy fix (although not necessarily the best if it turns out an update is required!) is to run a few commands in Terminal to add the new UUIDs[^1] to the SupportedPluginCompatibilityUUID key in the Info.plist.[^2]

**If you have already had your plugins disabled by opening Mail.app you'll need to look in ~/Library/Mail (or /Library/Mail if you installed globally) and move the files back to the active bundles directory.  They'll typically be in Bundles (Disabled), so quit Mail, find them, and move them back into the proper directory.**

Once you've gotten the files moved into the proper location use either the local installation or global installation commands below (depending on where you found your bundles).  To run them, simply copy/paste them into a Terminal window.[^3]

For GrowlMail (assuming local installation)

```bash
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "2F0CF6F9-35BA-4812-9CB2-155C0FDB9B0F"
defaults write ~/Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "0CB5F2A0-A173-4809-86E3-9317261F1745"
```

For GrowlMail (global installation)

```bash
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "2F0CF6F9-35BA-4812-9CB2-155C0FDB9B0F"
defaults write /Library/Mail/Bundles/GrowlMail.mailbundle/Contents/Info SupportedPluginCompatibilityUUIDs -array-add "0CB5F2A0-A173-4809-86E3-9317261F1745"
```


Update: [Letterbox has been updated](http://harnly.net/2009/software/letterbox/letterbox-beta-5-for-snow-leopard/#more-240), but the above instructions can be adapted for any future OS update that breaks compatibility.

**Update 2: You can [download](/assets/media/2009/11/GrowlMail.mailbundle.zip) a pre-patched copy if you don't want to follow the above instructions.  Just unzip it and drop it in your bundles directory.**

**Update 3: [GrowlMail 1.2.1](http://growl.info/growlmail/) has been released, which fixes 10.6.2 compatibility.  This issue will likely occur again with 10.6.3 though!**

[^1]: 2F0CF6F9-35BA-4812-9CB2-155C0FDB9B0F for Mail.app v4.2 and 0CB5F2A0-A173-4809-86E3-9317261F1745 for the Message framework.  These were released with OS X 10.6.2.

[^2]:  These UUIDs can be found in /Applications/Mail.app/Contents/Info.plist and /System/Library/Frameworks/Message.framework/Resources/Info.plist for when they inevitably change again.

[^3]: Terminal is found in /Applications/Utilities if you've never used it before.  You'll see a prompt and you can paste the commands in and hit enter.
