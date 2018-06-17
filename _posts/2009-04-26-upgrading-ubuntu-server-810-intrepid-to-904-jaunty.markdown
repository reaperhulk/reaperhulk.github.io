---
author: admin
comments: true
date: 2009-04-26 19:25:33+00:00
layout: post
slug: upgrading-ubuntu-server-810-intrepid-to-904-jaunty
title: Upgrading Ubuntu Server 8.10 (Intrepid) to 9.04 (Jaunty)
wordpress\_id: 558
tags:
- linux
- ubuntu
---

Now that Ubuntu 9.04 (Jaunty Jackalope) is out I decided to upgrade a few of my servers from 8.10 (Intrepid Ibex) to the latest release.  To upgrade via the command line follow these steps (as root or via sudo):


```bash
aptitude update
aptitude safe-upgrade
aptitude install update-manager-core
```

This will get your system upgraded to the latest packages of 8.10 and install the scripts to move to Jaunty.  Be sure you have screen installed as well -- Ubuntu does not like having upgrades interrupted so you want to be able to resume if your session is interrupted for any reason.

```bash
screen do-release-upgrade
```

It will download and check a few things and then warn you about operating an upgrade over ssh (along with starting an extra ssh daemon on port 9004).  Simply follow the instructions from there and you'll have a fully upgraded server![^1]


```bash
cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=9.04
DISTRIB_CODENAME=jaunty
DISTRIB_DESCRIPTION="Ubuntu 9.04"
```
[^1]: Bear in mind that during the upgrade you will occasionally be asked questions about config files, so don't try to run this upgrade unattended.
