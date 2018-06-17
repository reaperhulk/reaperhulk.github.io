---
author: admin
comments: true
date: 2010-06-12 05:14:58+00:00
layout: post
slug: ctrlswitcher-a-safari-5-extension
title: ctrlSwitcher, A Safari 5 Extension
wordpress\_id: 1162
tags:
- extension
- javascript
- safari
---

> [![](/assets/media/2010/06/ctrl-switcher-icon.png)](/assets/media/2010/06/ctrl-switcher-icon.png)
[View All My Safari Extensions](/safari-extensions/)

On the Ars Technica forums someone mentioned that they'd like to be able to switch between tabs using command + numbers to choose tabs.  I took a look at the Safari extension system, and while you can't override the shortcuts bound to cmd 1-9[^1] for some reason, control is available.  An hour or so later and ctrlSwitcher was born.

[![](/assets/media/2010/06/ctrlSwitcher-1.5-300x210.png)](/assets/media/2010/06/ctrlSwitcher-1.5.png)

> 
> ### Features
> 
> 

> 
> 

>   * Use a modifier (ctrl by default) + the number keys to instantly jump to a tab.  Keys 1 through 0 will go to tabs 1-10, and keys q through p will go to tabs 11-19.
> 

>   * Configurable modifier key
> 

>   * Configurable "go to last tab" key
> 



> 
> ### Download and Use
> 
> 

> 
> 

>   * **[Download](http://langui.sh/extensions/ctrlSwitcher.safariextz)** the signed extension and double click to install
> 

>   * Pick if you want to use ctrl, opt, or ctrl+opt as your meta keys (default ctrl) in the prefs
> 

>   * **Now close all tabs or restart your browser.** ctrlSwitcher has to load a small script in each loaded tab (empty tabs cannot be switched to/from due to limitations on extensions)
> 

>   * Note for Windows users: You will need to switch your default modifier key from ctrl to alt or ctrl+alt to have this tool work.


**v1.6:**

> 
> 

>   * Adds cmd+opt as a choice for key combo (see the [commit](https://github.com/reaperhulk/ctrlSwitcher/commit/e05407286cd82bdc9a18e49a26f43954e0e9e0fc))
> 

You can [view the source](http://github.com/reaperhulk/ctrlSwitcher) on GitHub as well!  If you have suggestions for improvements let me know!  **Bug reports should be directed to the [issues](http://github.com/reaperhulk/ctrlSwitcher/issues) page.**

[^1]: cmd-1 through 9 are assigned to bookmarks on your bookmarks bar in Safari
