---
author: admin
comments: true
date: 2009-04-22 02:50:22+00:00
layout: post
slug: ram-disks-in-os-x
title: RAM Disks in OS X
wordpress\_id: 547
tags:
- bash
- mac
---

If you've ever wanted to set up a ramdisk in Mac OS X, here's a quick script you can use.  Simply invoke it with a size in MB and the name for your volume and it will create and automount it for you.  It also won't leave any dangling directories in your /Volumes/ when you eject it!  Be sure to chmod the file you place the script in to make it executable (755 or likewise).


```bash

#!/bin/bash
if [ -n "$3" ]; then NUM_ARG=ERR; fi
if [ -z "$1" ]; then NUM_ARG=ERR; fi

if [ -n "$NUM_ARG" ];
then
echo 'Usage: ./ramdisk  <name, optional>'
exit
fi
SIZE=$1
NAME='my_ramdisk'
let "SIZE *= 2048"
if [ -n "$2" ]; then NAME=$2; fi
diskutil erasevolume HFS+ "$NAME" `hdiutil attach -nomount ram://${SIZE}`
echo 'Done'
```

