---
author: admin
comments: true
date: 2010-11-20 18:24:11+00:00
layout: post
slug: ocsp-queries-via-openssl
title: OCSP Queries Via OpenSSL
wordpress\_id: 955
tags:
- ocsp
- openssl
---

OpenSSL has an ocsp querying facility that can be useful if you're testing a responder or just curious how the online certificate status protocol works.  To use it:


```bash
openssl ocsp -issuer IssuingCert.txt -cert ServerCert.txt -url http://ocsp.wherever.com -CAfile CAchain.txt
```




### Argument Breakdown






  * -issuer is the issuing CA for the certificate you want to check (called IssuingCert.txt above).  This can be a self-signed root or a subroot. 


  * -cert is the certificate you want to verify.  If you know the serial number and don't want to provide the cert file itself you can use -serial instead.


  * -url is the URL of the OCSP responder for your cert.  You can parse the certificate to find the end point.  It will be under the Authority Information Access node inside the x509 extensions


  * -CAfile is only required if you want to verify the response of the OCSP server.[^1] You'll need to place the self-signed root + whatever intermediates are necessary for the OCSP signing cert from the server to chain up to it.


  * There are many other optional args, so check out the list just by typing "openssl ocsp"





### OCSP Response


Here's an example response where the certificate has been marked as revoked.

```
Response verify OK
ServerCert.txt: revoked
This Update: Nov 20 15:43:49 2010 GMT
Next Update: Dec  4 17:43:49 2010 GMT
Reason: unspecified
Revocation Time: Mar 31 21:37:52 2009 GMT
```

And one marked as acceptable.

```
Response verify OK
ServerCert.txt: good
This Update: Nov 20 11:20:51 2010 GMT
Next Update: Nov 27 11:20:51 2010 GMT
```


Responses can have several error status codes.  Here's the list of possible errors from [RFC 2560](http://www.ietf.org/rfc/rfc2560.txt).

```

malformedRequest      (1),  --Illegal confirmation request
internalError         (2),  --Internal error in issuer
tryLater              (3),  --Try again later
                            --(4) is not used
sigRequired           (5),  --Must sign the request
unauthorized          (6)   --Request unauthorized

```



[^1]: If you don't want to verify, use -noverify
