---
author: admin
comments: true
date: 2011-06-01 22:50:52+00:00
layout: post
slug: mongo-monitoring-via-mongo-live-in-safari-5
title: Mongo Monitoring Via Mongo Live In Safari 5
wordpress\_id: 1496
tags:
- github
- mongodb
- safari
---

We've recently started integrating MongoDB more extensively into our systems at work and found ourselves wanting some basic monitoring during dev to see what the DB was doing. A friend suggested [Mongo Live](https://github.com/deftlabs/mongo-live), which fit the bill but was only compatible with Chrome. Since I switch frequently between Safari and Chrome I decided to port it.

Fortunately the extension for Chrome required only minimal changes to get it working in Safari. The most significant change is that the Safari extension uses a toolbar button to activate. The fork is available on [Github](https://github.com/reaperhulk/mongo-live) and supports Chrome and Safari. Hopefully someone finds it helpful!
