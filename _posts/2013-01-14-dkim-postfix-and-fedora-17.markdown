---
author: admin
comments: true
date: 2013-01-14 16:57:54+00:00
layout: post
slug: dkim-postfix-and-fedora-17
title: DKIM, Postfix, and Fedora 17
wordpress\_id: 1861
tags:
- dkim
- email
- postfix
---

[DKIM](http://dkim.org), or DomainKeys Identified Mail, is a way to sign emails you send such that other mail servers can verify they came from your domain. This is accomplished by signing the email on the server with an RSA private key and having the receiving server verify the signature using a public key you publish via DNS TXT records. This can be very useful if you're sending large volumes of mail and want sites like gmail, yahoo, and hotmail to not treat you as spam. So, let's set it up for Fedora 17!



### Caveats


This guide is geared towards people who need to set up postfix to automatically sign outbound email for a specific domain. Extending it to sign mail for every domain on your server with a single key (or multiple different keys) is outside the scope, but is a trivial extension of the configuration described below.



### Install & Configure



```bash

yum install dkim-milter
cd /etc/mail/dkim-milter/keys
dkim-genkey -r -d whatever-your-domain-is.com

```

At this point you should have a "default.private" and "default.txt" file in your current working directory (which is /etc/mail/dkim-milter/keys). default.txt contains the DNS TXT record you **must add** to your DNS entries. Here's what it will look like:


```
default._domainkey IN TXT "v=DKIM1; g=*; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+rSlv01jhqLorJDSHe3hfbqFKMyRUdDKwh3BunLt8gBIsNvgoNfr1ykhjSA8jzgGc1n7zIXkI3RuoLHaXbGwb1kpuaWp+A4pfJz1plriem9qFRuCOYYkASy25UYK+MULYBwc73FoUQwGv7BvefCHNT6cQpOkSHaoJR4tcSL1OjwIDAQAB" ; ----- DKIM default for test
```


You'll also need to set up one more TXT record:


```
_domainkey IN TXT "t=y; o=~;"
```

The t=y line lets consumers know you are in test mode, and o=~ denotes that some (but not all) of your emails will be signed. When you're done testing be sure to remove t=y (and set o=- if you so desire).

Next let's rename and chown the private key so dkim-milter can use it.

```bash

mv default.private default
chown dkim-milter:dkim-milter default

```


Now we'll need to edit /etc/mail/dkim-milter/keys/keylist to add a line like this (substituting your domain):

```
*@whatever-your-domain-is.com:whatever-your-domain-is.com:/etc/mail/dkim-milter/keys/default
```


Finally, add the postfix group to the dkim-milter user in /etc/group and make sure /var/run/dkim-milter is set to 770 permissions.

We're all set configuring dkim-milter. Let's start it up and configure it to turn on at boot.

```bash

systemctl start dkim-milter.service
systemctl enable dkim-milter.service

```


Now that we've got dkim-milter set up we need to tell postfix to use it. Add the following lines to the end of your /etc/postfix/main.cf

```
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:/var/run/dkim-milter/dkim-milter.sock
non_smtpd_milters = local:/var/run/dkim-milter/dkim-milter.sock
```


Now restart postfix and we're ready to test

```bash
systemctl restart postfix.service
```




### Testing


To test, simply send an email from your server (using the domain you configured) to a gmail address. A valid signature will look like this valid signature from a google.com address:
![dkim-signed-email](/assets/media/2013/01/dkim-signed-email.png)
If you don't see a signed-by that means you've made a mistake in your configuration (or possibly your DNS hasn't had time to propagate properly). You can see what the error is by looking at the raw email headers. Good luck!
