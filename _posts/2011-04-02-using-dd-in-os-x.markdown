---
author: admin
comments: true
layout: post
slug: using-dd-in-os-x
title: Using dd in OS X
wordpress\_id: 1478
tags:
- bash
- dd
- diskutil
- mac
---

Doing device level copies with dd is a reasonably common Linux task, but not something OS X users typically do. However, if you find yourself needing to write an image file[^1] to a microSD card or some other media then here's a simple guide to using it.

First, you'll want to see the device labels.

```bash
diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *250.1 GB   disk0
   1:                        EFI                         209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            249.7 GB   disk0s2
/dev/disk2
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *2.0 GB     disk2
   1:                 DOS_FAT_32 THUMB_DRIVE             2.0 GB     disk2s1

```


The output from above shows that we have a mounted 2GB USB flash drive named "THUMB\_DRIVE" that has been given the identifier /dev/disk2. To write to it we need to unmount it, but ejecting it in the normal Mac way won't work here.


```bash
diskutil unmountDisk /dev/disk2
```


There we go! Now we just need to write our image file.


```bash
dd if=inputfile.img of=/dev/disk2
```


The shell will appear to hang, but you can check disk activity to see that it's writing. Once the command returns you're done!

[^1]: Frequently these guides will talk about using Win32DiskImager on Windows
