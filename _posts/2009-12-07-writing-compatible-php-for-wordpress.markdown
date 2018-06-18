---
author: admin
comments: true
layout: post
slug: writing-compatible-php-for-wordpress
title: Writing Compatible PHP For WordPress
wordpress\_id: 875
tags:
- php
- plugin
- wordpress
---

One of the big problems I've run into as a WordPress plugin developer is the diversity of PHP installations.  Simply stating you only support PHP5 and greater is insufficient to ensure compatibility.  Things you may take for granted in your development environment may be missing or worse, partially functional.  I've decided to document a few of the bigger "gotchas" I've run into so future WordPress hackers will have a starting point for investigating problems.



### Minimum PHP Version


One easy way to reduce compatibility confusion is specifying a minimum version of PHP for your WordPress plugin or widget. To do so you can use PHP's version\_compare function:

```php
if(version_compare(PHP_VERSION, '5.0.0','<')) {
	die(sprintf('You are currently running PHP version %s and you must have at least PHP 5.0.x', PHP_VERSION));
}
```

Take a look at the [version_compare documentation](http://us.php.net/manual/en/function.version-compare.php) for more information.


### PHP Modules


The biggest single thing to remember is that any module for PHP is optional.  If you want to require something like curl, that's great, but be aware that some non-trivial percentage of your potential users may not have it.  Still want to use that module?  Test for its presence during the instantiation of your plugin and die if it's not present.  This will prevent users from being able to successfully install your "broken" product if they don't have the proper requirements.

Sometimes you'll find yourself using functions you think are core to PHP like "mime\_content\_type".  This is not the case.  In fact, I have frequently run into PHP installations that have no MIME detection facilities at all.  Consider writing a fallback for suffix detection[^1] in addition to coding for mime\_content\_type and the finfo\_\* methods.



### Safe Mode and Related Problems


Another common issue is safe\_mode and open\_basedir. These flags enforce certain security restrictions (must have the same group as PHP when writing to a directory, can't open files beneath the basedir, et cetera).  However, they also impact some modules.  For instance, using curl you can't use CURLOPT\_FOLLOWLOCATION.  This will trigger the error "CURLOPT\_FOLLOWLOCATION cannot be activated when in safe\_mode or an open\_basedir is set".



### Network Communication


I've already talked a bit about the curl module, but you may also be tempted to use file\_get\_contents() assuming allow\_url\_fopen is enabled.  This is very frequently untrue in shared hosting environments.  Luckily, you don't need to rely on curl or allow\_url\_fopen or write your own fsockopen() wrapper class.  Instead, WordPress has integrated [Snoopy](http://sourceforge.net/projects/snoopy/) into the core.  Invoking it to fetch just the contents of a URL could not be easier!

```php
require_once( ABSPATH . 'wp-includes/class-snoopy.php');
$snoopy = new Snoopy();
$result = $snoopy->fetch($url);
if($result) {
	$response = $snoopy->results;
}
```

Just hit up Google for the class docs to learn everything Snoopy can do for you.

There are many more where these came from, so drop your problems and solutions in the comments!

[^1]: This can be a security risk in certain circumstances, so be careful.
