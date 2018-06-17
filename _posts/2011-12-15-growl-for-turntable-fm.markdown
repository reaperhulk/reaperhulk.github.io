---
author: admin
comments: true
date: 2011-12-15 01:54:35+00:00
layout: post
slug: growl-for-turntable-fm
title: Growl For turntable.fm
wordpress\_id: 1545
---

turntable.fm is a fun place to discover new music and play it for your friends, but it's frustrating to use at work since you have to switch to the tab to see what song is playing and who's saying what. I wrote a [Fluid](http://fluidapp.com) userscript that you can use to growl songs and chat lines to solve this problem. It also prevents idle timeouts from occuring in turntable if you don't interact with it for awhile.

```javascript

var debug = false;

var sendChat = false;

logger('loading userscript');

function bindDOM() {
    logger('binding');
    $('.messages').bind('DOMNodeInserted',parseChat);
    $(window).blur(function() {
        sendChat = true;
    });
    $(window).focus(function() {
        sendChat = false;
    });
}

function parseChat(event) {
    var message = event.target;
    var author = $(message).find('.speaker').text();
    var body = $(message).find('.text').text();
    if (body.indexOf('started playing') == 1) {
        var artist = body.match(/by (.*)/)[1];
        var song = body.match(/"(.+)"/)[1];
        logger('growling now');
        logger(artist);
        logger(song);
        showGrowl(artist,song);
    } else {
        body = body.substr(2);
        if (sendChat) {
            showGrowl(author,body);
        }
    }
}

function showGrowl(title,description) {
    window.fluid.showGrowlNotification({
        title: title, 
        description: description, 
        priority: 1, 
        sticky: false,
        identifier: "sh.langui.turntable"
    });
}

function idlePreventer() {
    logger('invoking idle preventer');
    $('.addSongsButton').click();
    $('.cancelButton').click();
}

function logger(data) {
    if(debug) {
        console.log(data);
    }
}

setTimeout(bindDOM,5000);
setInterval(idlePreventer,300000);

```


You'll need a licensed version of Fluid to use this userscript.
