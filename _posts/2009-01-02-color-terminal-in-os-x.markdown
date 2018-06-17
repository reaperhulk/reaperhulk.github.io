---
author: admin
comments: true
date: 2009-01-02 05:56:09+00:00
layout: post
slug: color-terminal-in-os-x
title: Color Terminal in OS X
wordpress\_id: 25
tags:
- bash
- mac
---

Apple ships OS X without ls set to use colors (and sans the useful ll alias).  For those of us who have grown to love the instant delineation between directories, symlinks, and files this is an inconvenience.  Luckily, adding this is quite simple.  Place the following into your ~/.profile and you're all set.  To change this setting for all users edit /etc/bashrc as a superuser, but be aware that future OS X updates could potentially erase your modifications.


```bash
export CLICOLOR=1
alias ll='ls -l'
```


If you'd like to alter the ls defaults for coloration you can add one more line:


```bash
export LSCOLORS=ExFxCxDxBxegedabagacad
```


The incomprehensible value is a custom color setting defined by [these rules](http://www.macgeekery.com/gspot/2007-01/interpreting_color_ls_output).

For those who are new to the command line, ~/.profile means a file named ".profile" in your home directory.  You can create the text file in TextEdit, save as plain text, then rename it to .profile, but be sure to move it into your home directory first as the Finder hides files prefixed with "." by default.
