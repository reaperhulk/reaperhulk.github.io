---
author: admin
comments: true
date: 2010-01-20 17:35:20+00:00
layout: post
slug: bash-productivity-enhancers
title: Bash Productivity Enhancers
wordpress\_id: 1043
tags:
- bash
---

Bash is an extremely powerful shell, but its shortcuts are not readily apparent.  Here are a few shortcuts and tips that I've noticed many (already proficient) bash users are not aware of.  You can also check out [Improved Bash History](http://langui.sh/2009/10/30/improved-bash-history/) and [More Useful Bash/Terminal Settings](http://langui.sh/2009/11/02/more-useful-bashterminal-settings/) for more ideas for improving your bash productivity.



## Bash Navigation Shortcuts


When editing a long command, there are quite a few navigation and editing shortcuts.  By default bash typically operates in emacs mode.




  * Ctrl-A to go to the beginning of the line


  * Ctrl-E to go to the end of a line.


  * Ctrl-W will cut the current word (searching backward)


  * Ctrl-U will cut everything before the cursor


  * Ctrl-K cuts everything after the cursor


  * Ctrl-Y pastes the last text that was cut


  * Ctrl-T swaps the order of the last two characters entered[^1]


  * Meta-B will move the cursor back one word


  * Meta-F will move the cursor forward one word


Meta keys are a bit tricky since they can differ based on your terminal application.  On Windows/Linux it is typically Alt and on Mac OS X Terminal.app defaults to using Esc (but you can change it to option/alt in the preferences).

However, bash also has a **vi/vim editing mode**.  To enable this type "set -o vi".  At this point all the typical vi shortcuts are available if you enter command mode (by hitting Esc).  I don't recommend using this unless you are very comfortable with vi already.



## reverse-i-search


You can search through your history and rapidly find a command used previously with reverse-i-search.  To invoke, press Ctrl-R and start typing.  If you have multiple matches, hit Ctrl-R to cycle through them all.  When coupled with an [improved bash history](http://langui.sh/2009/10/30/improved-bash-history/) this is an extraordinarily useful tool.



## Controlling Tasks in Bash


Bash allows you to stop, background, and foreground tasks.  To background a process before it starts simply add & to the end of your command.[^2]

```bash
mycommand &
[1] 1922
```


If you have an already running task and you'd like to stop it press Ctrl-Z.  This task will obtain a job number (the number in brackets).

```bash
[1]+  Stopped                 mycommand
```


You can then resume the task in the foreground with **fg #** or background it with **bg #**.   To see a list of jobs that have been backgrounded or stopped type **jobs**.



## Redirecting stderr/stdout in Bash


Bash has two main output buffers: stderr and stdout.  Both of these, by default, output to your terminal window.




  * To redirect stdout to a file add **> /path/to/output**


  * To redirect stderr to a file add **2> /path/to/output**


  * To redirect stderr into stdout add **2>&1**



[^1]: This shortcut is available in both emacs and vi mode, but I've placed it here since it uses the Control key.

[^2]: Output from stdout and stderr will continue to appear in your terminal, so consider redirecting them if needed.
