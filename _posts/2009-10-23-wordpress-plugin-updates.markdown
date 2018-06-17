---
author: admin
comments: true
date: 2009-10-23 22:30:29+00:00
layout: post
slug: wordpress-plugin-updates
title: WordPress Plugin Updates
wordpress\_id: 709
tags:
- cdn tools
- fidgetr
- plugin
- wordpress
---

I've spent quite a bit of time on [CDN Tools](/cdn-tools) and [Fidgetr](/fidgetr) in the past few weeks and this has cut back on the time I had planned to use to write blog entries.  I'll try to get a few new articles up soon, but in the mean time here is a status update on some projects you might be interested in...

CDN Tools (v0.9x and higher) is now compatible with WordPress 2.8+ and features a wide variety of reliability upgrades for various installation quirks.  I will be testing it shortly with WP 2.9 and expect to have a compatible version out prior to that release.  There are also some fun new features in the pipeline that will hopefully see the light of day in the next few weeks.

Regarding Fidgetr; I have decided to port the widget to the new WP 2.8 multi-widget API (which is adapted from the firetree multi-widget class).  While doing so I discovered that my previous assumptions about a single Fidgetr widget per WordPress page made porting quite difficult.  This necessitated an almost total rewrite of the core (and major modifications to the accompanying themes).  At this time I have the new widget mostly working, but there are many cosmetic bugs to resolve with the themes.  That said, I'm excited to offer this feature for those who desire it.  I have no targeted release date, but Fidgetr 2.0 will require WP 2.8+.  Fidgetr 1.3.5 is almost certainly the last 2.7 compatible release.
