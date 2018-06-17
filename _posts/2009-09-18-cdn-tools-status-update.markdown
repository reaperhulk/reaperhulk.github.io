---
author: admin
comments: true
date: 2009-09-18 16:15:42+00:00
layout: post
slug: cdn-tools-status-update
title: CDN Tools Status Update
wordpress\_id: 658
tags:
- cdn
- wordpress
---

I apologize to everyone who has been inquiring about when CDN Tools will be updated to add WP 2.8 compatibility.  I have been very busy lately and haven't found the time to get the issues resolved.  However, I am still planning to update it so don't lose hope.  In the meantime here are instructions to get CDN Tools working with 2.8 with only one issue that I'm aware of:





  1. Click remove files.  This will delete all JS and attachments from your CDN.


  2. Turn on advanced mode and click "load attachments"[^1]



Once you've completed these two steps the admin pages should be working properly and your attachments should be loading from the CDN again.  Side loading is also functional, although please be aware that deleting items does not work.  If you remove an attachment you will need to go to your CDN and manually remove it.

[^1]: If you click the normal "load files" CDN Tools will attempt to load both attachments and javascript.  The javascript load causes numerous issues in WP 2.8 with the admin pages.
