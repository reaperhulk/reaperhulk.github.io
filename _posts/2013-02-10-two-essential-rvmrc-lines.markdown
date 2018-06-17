---
author: admin
comments: true
date: 2013-02-10 21:22:14+00:00
layout: post
slug: two-essential-rvmrc-lines
title: Two Essential rvmrc Lines
wordpress\_id: 1936
tags:
- ruby
- rvm
---

If you use rvm on OS X and compile rubies more often than "almost never" you should really have a ~/.rvmrc file that contains these two lines.


```bash

cpus=`sysctl -n hw.ncpu`
rvm_make_flags="-j$cpus"

```

