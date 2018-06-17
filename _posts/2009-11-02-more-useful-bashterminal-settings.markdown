---
author: admin
comments: true
date: 2009-11-02 16:28:20+00:00
layout: post
slug: more-useful-bashterminal-settings
title: More Useful Bash/Terminal Settings
wordpress\_id: 751
tags:
- bash
- ssh
- terminal
---

A few more tricks to make your bash environment better.  As always, add them to your ~/.profile or ~/.bash\_profile to enable.

Disable the pagination of long lists when ambiguously tab completing.
```bash
bind 'set page-completions off'
```


Increase max returned items before being prompted.  (ie, "Display all 380 possibilities? (y or n").  You can set the number to whatever you'd like.

```bash
bind 'set completion-query-items 300'
```


Show the list of autocompletion options after the first tab.  This prevents the beep + second tab behavior.

```bash
bind 'set show-all-if-ambiguous on'
```


When autocompleting for cd or rmdir, list only directories as choices.

```bash
complete -d cd rmdir
```


Autocompletion for ssh known\_hosts.  Add this to your ~/.ssh/config (if the file doesn't exist, create it)

```bash
Host *
HashKnownHosts no
```


Make grep highlight the matching terms in its output.

```bash
export GREP_OPTIONS='--color=auto'
```


Ignore case for case preserving but insensitive filesystems (like HFS+).  I don't personally use this, but perhaps some people will like it.

```bash
bind 'set completion-ignore-case on'
```


Don't show hidden files when listing.  Another option I don't personally use.

```bash
bind 'set match-hidden-files off'
```

