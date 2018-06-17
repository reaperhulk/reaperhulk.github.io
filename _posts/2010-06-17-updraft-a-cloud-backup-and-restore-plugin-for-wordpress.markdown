---
author: admin
comments: true
date: 2010-06-17 17:54:51+00:00
layout: post
slug: updraft-a-cloud-backup-and-restore-plugin-for-wordpress
title: Updraft - A Cloud Backup and Restore Plugin For WordPress
wordpress\_id: 1036
tags:
- cloud
- plugin
- updraft
- wordpress
---

Over the past several months I have been working hard on a new plugin for WordPress and now I am pleased to unveil the initial release.

[Updraft](/updraft-wp-backup-restore/) is a backup and restore plugin for WordPress.  It can do scheduled or one time backups of your plugins, themes, uploads, and DB itself.  These backups can be kept locally, emailed, sent to an FTP server, uploaded to Rackspace Cloud Files, or transferred to Amazon S3.[^1]

Additionally, you can pick the number of backups to retain and restore from a backup of your choice either by using a local backup, uploading copies yourself, or pulling them down from your cloud service.[^2]

Visit the [Updraft homepage](http://langui.sh/updraft-wp-backup-restore) to learn more, see a screencast, and give feedback on bugs or features you'd like to see.  I look forward to hearing from you!

[^1]: Updraft requires WordPress 3.0 or greater!

[^2]: At this time restoration does not encompass DB.  You will need to download and restore the DB via another tool.  I am actively looking for ways to work around this issue for a future release.
