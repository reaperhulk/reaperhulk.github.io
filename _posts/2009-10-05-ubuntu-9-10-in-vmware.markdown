---
author: admin
comments: true
date: 2009-10-05 18:12:28+00:00
layout: post
slug: ubuntu-9-10-in-vmware
title: Ubuntu 9.10 In VMware - Updated
wordpress\_id: 694
tags:
- karmic koala
- linux
- ubuntu
- vmware
---

Update 2: Preliminary 10.04 instructions are available [here](/2010/02/27/ubuntu-10-04-in-vmware-fusion/). No real obstacles for those running the latest Fusion/Workstation.

Update: If you're using VMware Fusion 3.0 or any VMware Tools version 8.2.3-204229 or better you can follow a drastically simplified process.  **sudo apt-get install build-essential**, choose install VMware Tools from the menu, copy tar to desktop, untar, sudo ./vmware-install.pl and follow the instructions.  Simple!

With the release of the Ubuntu 9.10 RC it's time to revisit installing Ubuntu into a VMware VM.  I'm using VMware Fusion 2.0.x, but behavior should be largely the same for any recent VMware release.

First, create a new VM and point the installation disk at your Ubuntu 9.10 ISO.  At this time I would not recommend using "easy install", so uncheck that and continue.  If you wish to use the graphical installer you'll need to increase the RAM allocated to your VM from 512MB to 768MB.[^1]

Now you can boot your VM and follow the graphical installer.  Once complete your VM will hopefully reboot properly and ask you if you want to force the CD to disconnect (you do).  If this doesn't occur, force the guest to shut down, disconnect the ISO in the settings, then boot the VM again.

The official VMware Tools do not work properly due to the newer kernel (2.6.31), so we'll need to build the [open-vm-tools](http://sourceforge.net/projects/open-vm-tools/develop) for this kernel.  Follow the steps below to build them yourself or simply [download](http://c0396982.cdn.cloudfiles.rackspacecloud.com/open-vm-modules-2.6.31-14-generic_2009.07.22-179896-2+2.6.31-14.48_amd64.deb) the AMD64 deb package I have already built for the modules.[^2]




  1. Obtain the build prerequisities[^3]

```bash
sudo apt-get install open-vm-tools build-essential open-vm-toolbox
```



  2. Run module assistant to build the modules

```bash
sudo m-a
```



  3. Choose select and activate open-vm
[![Select open-vm](/assets/media/2009/10/Screen-shot-2009-10-25-at-12.37.06-PM-150x150.png)](/assets/media/2009/10/Screen-shot-2009-10-25-at-12.37.06-PM.png)


  4. Click okay, then select build
[![Screen shot 2009-10-25 at 12.41.35 PM](/assets/media/2009/10/Screen-shot-2009-10-25-at-12.41.35-PM-150x150.png)](/assets/media/2009/10/Screen-shot-2009-10-25-at-12.41.35-PM.png)
[![Screen shot 2009-10-25 at 12.37.37 PM](/assets/media/2009/10/Screen-shot-2009-10-25-at-12.37.37-PM-150x150.png)](/assets/media/2009/10/Screen-shot-2009-10-25-at-12.37.37-PM.png)


  5. Once it completes it will ask if you want to install.  Choose yes, then quit out of m-a and reboot.


  6. After a reboot check to see that the vm modules inserted into the kernel properly.

```
vmware@vmware-desktop:~$ lsmod | grep vm
vmsync                  5104  0
vmmemctl               10120  0
vmhgfs                 59080  0
vmci                   33952  0

```




If you see the 4 modules listed above then you should have functioning copy/paste, auto-resolution switching, and even shared folders.  However, to enable shared folders you'll need to follow these steps:


  1. Enable shared folders and add a folder in the VM settings[^4]


  2. Run this command:[^5]
```bash
sudo mount -t vmhgfs -v -o ro .host:/sharedfoldername /path/to/mnt
```



Let me know in the comments if you have issues or have improvements to the process.  Waiting for the official VMware Tools release is boring!

[^1]: It may not be necessary to increase it a full 256MB, but the 512MB default causes the install to fail as of the release candidate.

[^2]: If you choose to install the package, you'll need to do step 1 and then skip to step 6.

[^3]: open-vm-toolbox is only required for desktops

[^4]: If you get a message about "Unable to update run-time folder sharing status: The command is not recognized by the Guest OS tools" you can ignore this error.

[^5]: You can change ro to rw if you want your shared folder to be read/write capable
