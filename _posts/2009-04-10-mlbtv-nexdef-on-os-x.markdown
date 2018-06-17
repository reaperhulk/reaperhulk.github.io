---
author: admin
comments: true
date: 2009-04-10 21:31:15+00:00
layout: post
slug: mlbtv-nexdef-on-os-x
title: MLB.tv NexDef On OS X
wordpress\_id: 526
---

I have found myself growing tired of paying for cable TV, so this year I wanted to see if I could use MLB.tv as a replacement.  One of the major new features MLB is marketing for MLB.tv in 2009 is called "NexDef" and purports to provide an HD stream along with DVR functionality.  How could I go wrong?

As it turns out, NexDef runs as a daemon on both Windows and Mac (it uses Autobahn).  There is a shell app that launches at boot and invokes a Java http server.  Leaving aside the ugliness of the whole solution, having it running constantly in the background is beyond obnoxious.  Plus, even when you're not watching a game it will occasionally consume large quantities of CPU and bandwidth.[^1]

With that in mind I decided to tear apart the installation on OS X to find the jar it uses.  If you already have the app installed, check in ~/Library/Application Support/Swarmcast (that's your home directory's library directory) and find autobahn.jar.  Pull this out and then run the uninstall command located in the same directory to remove the launchd agent and other garbage it has placed on your system.  At this point you can now just invoke the autobahn.jar with Terminal.app.  As soon as the Jetty server comes up MLB.tv will detect it and everything will work.

```bash
java -jar /path/to/autobahn.jar
```

When you're done just hit ctrl-c in your terminal window and it will stop!

Update:  For those of you finding this page searching for Linux NexDef support, you can invoke the autobahn jar on the command line in Linux and (provided that you have Java 1.4.1 or later installed) it should work properly.

[^1]: Of course, Flash (even 10) on OS X is miserable, so NexDef at maximum quality takes all of one core on a 2.4GHz C2D MBP
