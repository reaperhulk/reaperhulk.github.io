---
author: admin
comments: true
date: 2009-01-08 22:53:27+00:00
layout: post
slug: filevault-performance-with-lightroom
title: FileVault Performance With Lightroom
wordpress\_id: 47
tags:
- camera
- lightroom
- mac
---

I have recently begun switching from Aperture to Lightroom for my photography workflow (look for another blog post about that at some point), but on my new unibody Macbook Pro I noticed that Lightroom appeared to be unbearably slow when dumping large volumes of data.  I suspected that FileVault (which I only keep enabled on my work laptop) was the culprit, so I decided to perform a few benchmarks.


### Test Setup


The import was performed using a Sandisk Extreme USB 2.0 CF Reader with a Lexar 8GB UDMA card.  There were 113 21MP RAW files (beach photos from a 5D Mark II) on the card totaling 2.61GB of data.  Lightroom's import was set to copy into the home directory and generate 1:1 previews after import.  I used a unibody MBP running at 2.4ghz, 250GB 5400RPM drive, and 2GB of RAM with two separate user accounts (FV on and FV off).


### Results


[](/assets/media/2009/01/fv_perf1.png)[![FV Performance in Lightroom](/assets/media/2009/01/fv_perf.png)](/assets/media/2009/01/fv_perf.png)


As you can see, while generating 1:1 previews varies by less than 2%, the initial import takes nearly twice as long.  Additionally, during the import to a FileVault enabled home directory the machine is nearly unusable due to heavy disk I/O.  diskimage-helper hovers around 40-60% CPU during this time.

Of course, 321 seconds for a 2.61GB import is not particularly fast.  Let's take a look at 4 transfers of the same data to see what types of speed we're getting.

[![RAW Copy Performance](/assets/media/2009/01/raw_data1.png)](/assets/media/2009/01/raw_data1.png)

Import performance is essentially equal with FileVault both on and off when simply copying files via the Finder (~17MB/sec with this USB 2.0 reader).  So then, FileVault doesn't affect Finder copy performance...why is it affecting Lightroom?


### Digging Deeper


One of the side effects of FileVault's method of home directory encryption is that any file on the hard drive outside of the encrypted home directory must be re-copied (rather than simply moved) if you wish to move it into home.  Using [fseventer](http://www.fernlightning.com/doku.php?id=software:fseventer:start) we can track where Lightroom is placing files during import, and it appears that significant quantities of data are being stored in /tmp and then copied into the home directory when the file is done.  This is a very expensive operation compared to a direct copy when FV is enabled.


### Outstanding Questions


I'm still a bit baffled as to why Lightroom is so much slower on import vs a Finder copy even when FV is off.  A move operation is essentially free when compared to the expense of a full copy, so if Lightroom copies to /tmp and then moves why isn't FV off nearly as fast as a direct copy?  Obviously LR has quite a few operations (minimal thumbnails, catalog updates, et cetera) to perform, but does that account for the twofold drop in data transfer speed?  Is there something else I'm missing?  I'll revisit this issue with Aperture performance numbers and perhaps data points from an older Santa Rosa Macbook Pro.
