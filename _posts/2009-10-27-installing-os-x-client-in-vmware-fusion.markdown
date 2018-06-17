---
author: admin
comments: true
date: 2009-10-27 15:08:47+00:00
layout: post
slug: installing-os-x-client-in-vmware-fusion
title: Installing OS X Client in VMware Fusion
wordpress\_id: 722
tags:
- mac
- vmware
---

Mac OS X's EULA only allows for usage of OS X Server within VMware Fusion, but with a (relatively) simple script you can modify the darwin.iso to function with OS X client as a guest.[^1] This script should hypothetically work with Fusion 2.0 and 3.0 on Leopard and Snow Leopard (as both guest and host).  Save the following script to a file.

```bash
#!/bin/bash
cd "/Library/Application Support/VMware Fusion/isoimages"
mkdir original
mv darwin.iso tools-key.pub *.sig original
perl -n -p -e 's/ServerVersion.plist/SystemVersion.plist/g' < original/darwin.iso > darwin.iso
openssl genrsa -out tools-priv.pem 2048
openssl rsa -in tools-priv.pem -pubout -out tools-key.pub
openssl dgst -sha1 -sign tools-priv.pem < darwin.iso > darwin.iso.sig
for i in *.iso ; do openssl dgst -sha1 -sign tools-priv.pem < $i > $i.sig ; done
exit
```

Now open Terminal and chmod the script to executable.

```bash
chmod 755 /path/to/my/script
```

Finally, execute the script with root privileges.

```bash
sudo ./path/to/my/script
```

This will modify and re-sign the darwin.iso to allow OS X client as a guest.  Hat tip to several sources online (which I can no longer remember) that were used to help make this script many moons ago.

[^1]: Of course, since this is against the license agreement no one is going to use it, right?
