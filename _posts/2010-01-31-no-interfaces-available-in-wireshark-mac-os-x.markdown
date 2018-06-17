---
author: admin
comments: true
date: 2010-01-31 22:36:11+00:00
layout: post
slug: no-interfaces-available-in-wireshark-mac-os-x
title: No Interfaces Available In Wireshark Mac OS X
wordpress\_id: 1082
tags:
- mac
- wireshark
---

Many new Wireshark users on Mac OS X run into an issue where no interfaces show up when trying to begin packet capture.  If you attempt to manually input an interface (such as en0) this error will occur:


> The capture session could not be initiated ((no devices found) /dev/bpf0: Permission denied).



To have the interfaces show up properly you'll need to widen the permissions on the Berkeley packet filter (BPF).  By default they look like this:

```bash
crw-------  1 root  wheel   23,   0 Jan 31 13:47 /dev/bpf0
```

We need the filter to be readable by non-root, so open Terminal.app and run this command:

```bash
sudo chmod 644 /dev/bpf*
```


Unfortunately every time you reboot this will reset, but if you are a frequent user of Wireshark you can add the ChmodBPF StartupItem to alter them automatically (available in the Utilities folder on the Wireshark disk image).  To install you'll need to follow two steps.

First, drag the ChmodBPF folder to the StartupItems alias in the same folder (or drag it to /Library/StartupItems directly).  Type your password to authenticate and move the folder into the correct location.

The second requirement is only for 10.6+ users.  Starting with Snow Leopard the security permissions of StartupItems are being enforced.  Scripts that do not have the proper owner and group will receive this error:


> Insecure Startup Item disabled. – “/Library/StartupItems/ChmodBPF” has not been started because it does not have the proper security settings


The proper security settings are ownership of the scripts by root and group of wheel.[^1]  To set them:

```bash
sudo chown -R root:wheel ChmodBPF
```



[^1]: The correct settings for startup items can be found in this [Apple KB article](http://support.apple.com/kb/HT2413)
