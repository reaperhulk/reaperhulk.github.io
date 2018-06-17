---
author: admin
comments: true
date: 2009-02-23 00:07:57+00:00
layout: post
slug: code-signing-for-mac-os-x-and-windows
title: Code Signing for Mac OS X and Windows
wordpress\_id: 112
tags:
- codesign
- crypto
- openssl
- ssl
---

Code signing is rapidly becoming an important part of application deployment on many platforms.  On OS X it suppresses the keychain warnings when you update your application and on Windows it can bypass numerous UAC notifications as well as the initial application launch dialog.  This can (sometimes drastically) improve the customer experience and reduce friction associated with your application.  But how do you actually do it?

You can purchase a code signing certificate from any major CA, but for today we're going to use the [OpenSSL Self-Signed CA](/2009/01/18/openssl-self-signed-ca/) we created in a previous article.

First let's create a code signing certificate (if you purchased a certificate you will not need to perform these steps):

```bash
 openssl req -newkey rsa:1024 -nodes -out codesign.csr -keyout codesign.key
```

This will write out a codesign.key and codesign.csr after you choose the fields you desire.  Since we are making a code signing certificate the common name should be your company name.  You can also leave the challenge password blank.

```
Generating a 1024 bit RSA private key
..........++++++
..................++++++
writing new private key to 'req.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [GB]:US
State or Province Name (full name) [Berkshire]:Illinois
Locality Name (eg, city) [Newbury]:Chicago
Organization Name (eg, company) [My Company Ltd]:End Entity, Inc.
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:End Entity, Inc.
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

Now that we've created the CSR, we need to modify the myca.conf file to flag certificates for code signing.  Find this line

```
extendedKeyUsage = serverAuth
```

and change it to

```
extendedKeyUsage = codeSigning
```

Now we're all set to sign the certificate.

```bash
 openssl ca -batch -config /path/to/myca.conf -notext -in codesign.csr -out /path/to/codesign.cer
```

At this point we should have a codesign.cer and a codesign.key file and we're ready for the actual signing process.


### Code Signing on OS X


In OS X the codesign binary is used to sign applications.  Codesign cannot accept the private key/certificate pair as a command line parameter, so you must place them in your Keychain.  To do this you need to convert your CER + RSA private key into a PKCS12 (PFX) file.  Luckily there's already a post here about that, so check out [Generating a PKCS12 (PFX) Via OpenSSL](/2009/01/24/generating-a-pkcs12-pfx-via-openssl/) for more information.
Once you have it in the proper format, simply double click to import.  Now to sign your application:

```bash
codesign -s 'End Entity, Inc.' Whatever.app
```

Inside the single quotes you'll want to choose the name (CN) of the certificate as seen in Keychain Access.
If you chose to sign the application with a self-signed cert, please note that OS X won't trust it unless you add the created root to your Keychain.  If you do not you will receive this warning:

```
Whatever.app/: CSSMERR_TP_NOT_TRUSTED
```




### Code Signing XP/Vista/Windows 7


To code sign in Windows you should have an SPC (aka PKCS7/P7B) or CER file along with a pvk formatted private key.  To take a standard RSA private key and convert to PVK you should use [PVK Tool](http://www.drh-consultancy.demon.co.uk/pvk.html) (Download link is under the conversion tools heading)
Convert your private key to PVK format

```bash
pvk.exe -topvk -nocrypt -in codesign.key -out codesign.pvk
```

Once you have the files in the format you require execute [SignTool](http://msdn.microsoft.com/en-us/library/aa387764.aspx) from a windows command prompt.

```bash
 signtool.exe signwizard
```
[![wizard1](/assets/media/2009/02/wizard1-150x150.png)](/assets/media/2009/02/wizard1.png)
[![wizard2](/assets/media/2009/02/wizard2-150x150.png)](/assets/media/2009/02/wizard2.png)
[![wizard3](/assets/media/2009/02/wizard3-150x150.png)](/assets/media/2009/02/wizard3.png)
[![wizard4](/assets/media/2009/02/wizard4-150x150.png)](/assets/media/2009/02/wizard4.png)
[![wizard5](/assets/media/2009/02/wizard5-150x150.png)](/assets/media/2009/02/wizard5.png)
[![wizard6](/assets/media/2009/02/wizard6-150x150.png)](/assets/media/2009/02/wizard6.png)
[![wizard7](/assets/media/2009/02/wizard7-150x150.png)](/assets/media/2009/02/wizard7.png)
[![wizard8](/assets/media/2009/02/wizard8-150x150.png)](/assets/media/2009/02/wizard8.png)
[![wizard9](/assets/media/2009/02/wizard9-150x150.png)](/assets/media/2009/02/wizard9.png)
[![wizard10](/assets/media/2009/02/wizard10-150x150.png)](/assets/media/2009/02/wizard10.png)
[![wizard11](/assets/media/2009/02/wizard11-150x150.png)](/assets/media/2009/02/wizard11.png)
[![wizard12](/assets/media/2009/02/wizard12-150x150.png)](/assets/media/2009/02/wizard12.png)
[![wizard13](/assets/media/2009/02/wizard13-150x150.png)](/assets/media/2009/02/wizard13.png)
[![wizard14](/assets/media/2009/02/wizard14-150x150.png)](/assets/media/2009/02/wizard14.png)

And we're done!  If you chose to sign the application with a self-signed cert, please note that Windows won't trust it unless you add the created root to your certificate store.
