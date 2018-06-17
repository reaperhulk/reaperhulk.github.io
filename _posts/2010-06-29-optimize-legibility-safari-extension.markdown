---
author: admin
comments: true
date: 2010-06-29 16:23:29+00:00
layout: post
slug: optimize-legibility-safari-extension
title: Optimize Legibility (Safari Extension)
wordpress\_id: 1318
---

[![](/assets/media/2010/06/Icon-641.png)](/assets/media/2010/06/Icon-641.png)
[View All My Safari Extensions](/safari-extensions/)

[John Gruber](http://daringfireball.net) linked to an [article](http://www.aestheticallyloyal.com/public/optimize-legibility/) about the text-rendering:optimizeLegibility CSS property today and as he said, news to me.  I've built a quick Safari extension to enable it by default in Safari (Firefox already does this).  So, without further ado:

Optimize Legibility is a simple Safari extension that injects a single CSS attribute (text-rendering:optimizeLegibility) into every page.  This will improve kerning and ligatures in text.

[Download](http://langui.sh/extensions/Optimize-Legibility.safariextz) it or [view the (trivial) source](http://github.com/reaperhulk/Optimize-Legibility) at Github.

Update: 1.0.2 adds an icon and reverts the CSS to matching body rather than wildcard. Should improve performance (although it won't be noticeably faster on any modern machine).
