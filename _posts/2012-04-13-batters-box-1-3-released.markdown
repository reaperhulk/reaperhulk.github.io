---
author: admin
comments: true
date: 2012-04-13 16:52:59+00:00
layout: post
slug: batters-box-1-3-released
title: Batter's Box 1.3 Released
wordpress\_id: 1652
---

Apple approved [Batter's Box](http://itunes.apple.com/us/app/batters-box/id510063319?ls=1&mt=8) 1.3 last night. The biggest single change is that the iPhone details view now has all the same graphics as the iPad version. Here's a screenshot:
[![](/assets/media/2012/04/photo-2-200x300.png)](/assets/media/2012/04/photo-2.png)

You can also see more screenshots [here](/batters-box).



### Complete Changelog




  * iPhone now has the same view as the iPad on the detail page. Including weather, day/night, and fireworks.


  * iPhone now shows tv/radio station information prior to game when you tap a game.


  * Preview link now shows on the game list for each game before first pitch


  * Last play always shows for in progress games on iPad (+some animations)


  * Walks are now properly recognized as the end of an at bat (probably still have bugs with wild pitch reach base or other obscure situations)


  * Fix rain animation so it triggers properly when a rain delay occurs (iPad)


  * Preview link text no longer gets truncated


  * Animate batter when switching between L/R hitters


  * Handle "Dome" weather condition


  * Fix case where AT&T; Park was rendered as AT&T Park


  * Nicer loading spinner for highlights view


  * The data webview on the game details page now maintains scroll position across reloads


  * Reload timers now disable when you leave the app and re-enable when you foreground it. Any loads in progress is canceled when going to background.


  * Better handling of long idle times in the background


  * Reworked underlying logic for games view controller. The 3 game cell layouts are now separate prototypes and classes now to improve code readability



I'll be submitting 1.4 to Apple shortly with another batch of new features (and a few bug fixes for small visual issues that are present in the 1.3 build). Enjoy!
