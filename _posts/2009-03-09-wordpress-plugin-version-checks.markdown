---
author: admin
comments: true
date: 2009-03-09 04:58:12+00:00
layout: post
slug: wordpress-plugin-version-checks
title: WordPress Plugin Version Checks
wordpress\_id: 384
tags:
- wordpress fidgetr
---

I recently noticed that in certain cases WordPress was not providing notifications that an upgrade was available for my plugins.  This, naturally, irritated me since I define myself by how many downloads I receive per day.[^1]  So, to regain my self-esteem I decided to dig a bit to find the root cause.[^2]

To determine if a plugin is updated WordPress sends a list of installed plugins and their version numbers to a script (update-check) hosted on api.wordpress.org.  While the source code for this script is not publicly available, it almost certainly uses PHP's version\_compare function to determine when a plugin has changed.  Accordingly, I did some tests with some of the version numbers of Fidgetr to see what was going on...


### Version Comparison Tests


* 0.32 > 0.4
* 0.32 > 0.6.1
* 0.32 < 0.41
* 0.32 < 1.0
* 0.32 < 0.51
* 0.51 < 1.0

This is just a small subset, but you can see that version\_compare expects the x.x.x format and does not handle x.xx properly.  Since I have done quite a few Fidgetr releases using the x.xx versioning there are potentially hundreds of outdated installations not receiving proper version updates.

Due to this issue, the next revision of Fidgetr will be 1.0 since updating the major version corrects the problem.  I would prefer not to call it 1.0 without a big update, but ensuring that all of Fidgetr's users can obtain proper updates takes precedence.

Hopefully documenting this here will prevent some headaches for other plugin developers.

[^1]: This is a lie.  It's actually downloads per week.  I don't have fine enough time resolution for per day monitoring.

[^2]: It seemed like a better idea than eating 5 boxes of Thin Mints...although that was a close second.
