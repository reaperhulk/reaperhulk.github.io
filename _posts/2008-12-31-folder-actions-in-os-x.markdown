---
author: admin
comments: true
date: 2008-12-31 20:23:43+00:00
layout: post
slug: folder-actions-in-os-x
title: Folder Actions in OS X
wordpress\_id: 17
tags:
- applescript
- mac
---

Awhile back I started getting annoyed at the process of manually moving files around after downloading, so I decided to investigate one of Apple's more hidden features...Folder Actions!  For those who don't know, folder actions are configurable scripts that can be attached to folders and run whenever the contents of the folder change. They can do anything from copying mp3s you download from Amazon MP3 automatically into iTunes to converting documents automatically into PDF.

In a fit of boredom I have made a [screencast](/assets/media/2008/12/folderactions.mov) showing how to make a simple folder action. If you'd like to follow along yourself, here is the applescript script (please note that you'll need to create a folder named JPEGs in your ~/Downloads folder as well).  Open Script Editor and paste it in to begin!


```applescript
on adding folder items to my_folder after receiving the_files
tell application "Finder"
set jpgFolder to folder "JPEGs" in my_folder
repeat with f in the_files
if f's name ends with ".jpg" then
move f to jpgFolder
end if
end repeat
end tell
end adding folder items to
```


Obviously this represents a profoundly simple use case, but folder actions can have near limitless complexity.  Unfortunately there are several issues with the implementation of this feature in OS X.  Whenever the folder action actually triggers you will briefly get a flicker in your foreground window as the Finder takes focus, and for reasons I can't quite nail down actions do not seem to trigger with 100% reliability.  Despite these issues, this is a powerful tool that can be exploited in fascinating ways.
