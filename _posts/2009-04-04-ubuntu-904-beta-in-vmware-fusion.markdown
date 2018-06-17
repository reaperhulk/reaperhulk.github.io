---
author: admin
comments: true
date: 2009-04-04 23:42:56+00:00
layout: post
slug: ubuntu-904-beta-in-vmware-fusion
title: Ubuntu 9.04 (beta/RC/final) In VMware Fusion
wordpress\_id: 486
tags:
- linux
- ubuntu
---

**Update 10/26/09** Ubuntu 9.10 (Karmic Koala) is about to be released, so check out the [new guide](/2009/10/05/ubuntu-9-10-in-vmware/) for that version.

**Update 4/25/09** Ubuntu 9.04 (Jaunty Jackalope) final was released a few days ago so I grabbed it and rebuilt my VM.  I don't recommend using the "easy install" option at this time, so uncheck that and run through the Ubuntu installer by hand.  There are only 7 steps and you can leave everything default if you want.  Once installed you can follow the instructions below (including the vmmouse install) to get it all up and running.

**Update 4/21/09:** RC is now out and to get mouse ungrab working you can simply run this command:

```bash
aptitude install xserver-xorg-input-vmmouse
```

Be sure to aptitude update to get the latest package lists.  I'll do a new post when Ubuntu 9.04 is released in a few days.  Be sure to check the comments of this post for other takes on getting this working properly!


Now that Ubuntu 9.04 has gone into beta I decided to build up a quick VM for random testing purposes.  Installation is very straightforward: simply download the x86 or AMD64 CD image and install.  However, since this is a beta things are in heavy flux -- I would highly recommend upgrading to the latest packages before installing the VMware Tools since several bugs (like the infinitely multiplying window when mounting the VMWare Tools image) have been fixed.

Select "Install VMware Tools" from the Virtual Machine menu and then copy the tar to your Ubuntu desktop.  Running the vmware-install.pl as superuser will net you a mostly functional system, but drag/drop files and mouse ungrab will not work.[^1]

To get VMHGFS working (drag/drop file sharing), you will need to follow these instructions:




  1. Navigate to vmware-tools-distrib/lib/modules/source and untar vmhgfs.tar.  Inside the vmhgfs-only directory that is created there is a file called page.c.


  2. Edit this file and do the following[^2]

  ```diff
  -    page = __grab_cache_page(mapping, index);
  +    page = grab_cache_page_write_begin(mapping, index, flags);
  ```



  3. Move the vmhgfs.tar file to vmhgfs-old.tar and create a new tar with the modified file

  ```bash
  tar cvf vmhgfs.tar vmhgfs-only
  ```


  4. Run the vmware-install.pl script (as superuser)



At this time I haven't managed to get mouse ungrab working, but let me know if you figure out a way!  The vmmouse driver for X.org is available via apt-get install xserver-xorg-input-vmmouse, but does not appear to work any better than the one that comes with VMware Tools 7.9.3.  Hopefully a driver compatible with X.org 7.5 shows up soon.

[^1]: You may need to run sudo apt-get install build-essential to get the basic build environment. I don't remember if I did this when building mine.

[^2]: For those unfamiliar with diff syntax, remove the first line and replace it with the second line.
