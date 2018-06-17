---
author: admin
comments: true
date: 2010-11-14 19:30:28+00:00
layout: post
slug: pbpaste-pbcopy-in-mac-os-x-or-terminal-clipboard-fun
title: 'pbpaste & pbcopy in Mac OS X (or: Terminal + Clipboard = Fun!)'
wordpress\_id: 758
tags:
- bash
- os x
---

The OS X shell is very powerful, but some wonderfully useful commands are almost entirely unknown to the community at large.  Two of these forgotten commands are pbcopy and pbpaste. Let's take a quick look at what they can do.



## pbcopy


This command allows you to copy text from stdin into the clipboard[^1] buffer.  Trivial example:

```bash
echo 'Hello World!' | pbcopy
```

"Hello World!" is now in your clipboard.



## pbpaste


Pastes from your clipboard to stdout. Trivial example:

```bash
echo `pbpaste`
```

This will echo the contents of your clipboard.  If you're following along you'll see "Hello World!".



## What Can I Do With These?


What can't you do!  Oh, you want examples?  Well...




  * You could grab the output of a grep/awk/sed to paste into IM/IRC.


  * You could use a macro tool (like iKey, QS, et cetera) to create text modifying workflows that grab highlighted text, manipulate it, and replace it inline.


  * You could pull changelogs from svn into the clipboard when tagging for release so you could email them to coworkers.


Let me know what amazing things you come up with to enhance your own productivity!

[^1]: or pasteboard, hence the prefix "pb"
