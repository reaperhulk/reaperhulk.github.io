---
author: admin
comments: true
date: 2012-04-17 04:02:23+00:00
layout: post
slug: ruby-opensslx509nameto_a-followup
title: Ruby OpenSSL::X509::Name#to_a Followup
wordpress\_id: 1660
---

Awhile back I [talked about an issue](/2011/12/21/ruby-opensslx509nameto_a-dissection/) (and workaround) regarding how Name#to\_a worked in Ruby 1.9.3p0 and earlier. I'm happy to report that my [patch](http://bugs.ruby-lang.org/issues/5787) was accepted (and rewritten by core devs to be better!) and subsequently [backported](http://bugs.ruby-lang.org/issues/5983) to Ruby 1.9.3. That fix was released as part of the 1.9.3-p125 release in February. Pretty quick turnaround!
