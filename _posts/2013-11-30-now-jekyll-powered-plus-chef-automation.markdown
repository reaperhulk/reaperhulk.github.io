---
layout: post
title: Now Jekyll Powered (Plus Chef Automation)
---
Recently I realized that while I write extensive automation at work, my own servers are set up by hand. Coupled with the announcement of SSD-based cloud servers from Rackspace I realized I had a golden opportunity to rectify this lack.

Of course, it's never that simple. My blog has been running WordPress for several years and I've used a variety of plugins (galleries, footnotes, syntax highlighting), all of which have their own syntactical quirks. For example, I had to write a parser to find each footnote and translate it to a footnote format that was compatible with the new markdown parser.

In addition to the blog itself I also run several other sites such as [SAS Eliminator](http://saseliminator.com), which is a Ruby app proxied by nginx with a MariaDB backend.

Working through the issues and constructing the automation took quite a bit of time, but I'm happy to report that it is complete! After researching [Ansible](http://www.ansibleworks.com) for several hours I ended up going with [Chef](http://opscode.com) because the initial server creation was more intuitive (and I admittedly already knew something about Chef).

Using the [knife-rackspace](https://github.com/opscode/knife-rackspace) plugin and [hosted chef](https://manage.opscode.com/login) I can now boot two VMs and bootstrap them to full configuration in just a few minutes. There are still several pieces of manual work that I'll be spending time on eliminating over the next few months, but I'm pleased for now!
