---
author: admin
comments: true
date: 2009-10-30 19:45:12+00:00
layout: post
slug: improved-bash-history
title: Improved Bash History
wordpress\_id: 732
tags:
- bash
---

If you use multiple shells simultaneously (in my case with Terminal.app on OS X) you've undoubtedly noticed that the history of the last closed shell clobbers any commands you might have executed in others.  This makes it difficult to use reverse-i-search to find commands you recall using.  However, with a few modifications to your bash history you can greatly increase its utility.

```bash
export HISTCONTROL=erasedups
export HISTSIZE=10000
export HISTTIMEFORMAT="%D %T "
export HISTIGNORE="&:ls:exit"
shopt -s histappend
```

Save the above lines to your home directory's .profile (or .bash\_profile) and your shell history will now prevent duplicates, have a maximum of 10,000 items, append a timestamp to all new commands, exclude a list of commands, and append history between shells.
