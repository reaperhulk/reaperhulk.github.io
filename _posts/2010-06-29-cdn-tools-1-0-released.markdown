---
author: admin
comments: true
date: 2010-06-29 14:42:49+00:00
layout: post
slug: cdn-tools-1-0-released
title: CDN Tools 1.0 Released
wordpress\_id: 1273
---

After a long delay I'm proud to announce the release of [CDN Tools](/cdn-tools/) 1.0.  This a major release that requires you to reload all your sideloaded files.  **Do not upgrade to 1.0 from 0.99 unless you're ready to reload everything!**

New Features:




  * WP 3.0 compatibility[^1]


  * Changed method of storing info about sideloaded files to be far more robust.

  * Use directory structure on Cloud Files (uses just one container now)


  * Now CDNifies post thumbnails as well (WP 2.9+ feature)


  * Fix for blogs using SSL


  * Caches credentials for more rapid initial loads/multiple media attach uploads.


  * You can now define constants in wp-config for plugin configuration. This allows to configure settings that will be active on all end-user sites without allowing them to see/edit the config. If you define constants the CDN Tools admin page will not register a configuration page.



Head over to my [CDN Tools](/cdn-tools/) page to learn more or go to wordpress.org to [download](http://wordpress.org/extend/plugins/cdn-tools/) it now.

[^1]: 0.99 probably works with 3.0 as well
