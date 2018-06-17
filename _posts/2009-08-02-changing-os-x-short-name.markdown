---
author: admin
comments: true
date: 2009-08-02 16:05:02+00:00
layout: post
slug: changing-os-x-short-name
title: Changing OS X Short Name
wordpress\_id: 637
tags:
- mac
---

When my work laptop was originally set up the short name was set to "paul kehrer".  This space tends to cause issues with poorly written applications, especially on the command line, so I set out to change it.  Prior to Leopard this was a difficult task best accomplished with [ChangeShortName](http://www.danfrakes.com/ChangeShortName.html), but with Leopard you can now do it without any third party utilities.

[![advancedoptions](/assets/media/2009/08/advancedoptions1.png)](/assets/media/2009/08/advancedoptions1.png)First you'll need to create a secondary admin account and log into that.  While you can directly change the short name of an account you are logged into, I wouldn't recommend it.  Next, open system preferences, go to accounts, and right click the user you want to change.  You'll see a single menu item called "advanced options...".

[![useroptions](/assets/media/2009/08/useroptions-150x150.png)](/assets/media/2009/08/useroptions.png)Choosing that will bring up a sheet that lets you change your user ID, group ID, short name, logins hell, home directory path, UUID, and add/remove aliases (other login names) for your account.  While these are all very powerful tools we're only going to use short name and home directory this time.  In general it is safest to keep your short name and home directory path synchronized to avoid any issues that might arise from poorly written apps, but it is not required to change both.  If you do change the home dir, then be aware that you'll also need to run this command from the terminal to complete your account switch.

```bash
sudo mv /Users/oldshortname /Users/newshortname
```

Once you've done this you can log back in to the other account and you're all set.  Be aware that some apps do use hard coded paths for preferences, so you'll probably have a few small issues to correct as you get up to speed.
