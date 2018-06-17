---
author: admin
comments: true
date: 2011-12-21 21:54:31+00:00
layout: post
slug: ruby-opensslx509nameto_a-dissection
title: Ruby OpenSSL::X509::Name#to_a Dissection
wordpress\_id: 1559
tags:
- openssl
- ruby
---

Over at [Viking Hammer](http://vikinghammer.com) my coworker Sean Schulte has written up a [great article](http://vikinghammer.com/2011/12/21/ruby-opensslx509name-throws-away-unknown-subject-component-names/) dissecting an issue we ran into regarding the way Ruby's OpenSSL::X509::Name#to\_a currently builds its array. He discusses the problem, the two solutions we came up with, and shares code examples. Go check it out!
