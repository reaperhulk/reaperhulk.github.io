---
author: admin
comments: true
date: 2009-02-27 23:45:52+00:00
layout: post
slug: creating-a-subjectaltname-sanucc-csr
title: Creating a SubjectAltName (SAN/UCC) CSR
wordpress\_id: 318
tags:
- crypto
- openssl
- ssl
---

SAN certificates (or as Microsoft and others have taken to calling them, Unified Communications Certificates) are rapidly becoming a popular option for securing multiple domains.  In fact, Exchange 2007, OCS 2007, and several other products now require UCC to function.  However, this certificate type can proffer some advantages beyond that of a wildcard certificate as well.  One such advantage is the ability to secure "domain.com", "www.domain.com", "domain.net", and "someotherdomain.com" all within a single certificate.

SAN CSRs cannot be generated using the interactive prompt in OpenSSL so we'll need to make a conf:

```
[ req ]
default_bits        = 1024
default_keyfile     = privkey.pem
distinguished_name  = req_distinguished_name
req_extensions     = req_ext # The extentions to add to the self signed cert

[ req_distinguished_name ]
countryName           = Country Name (2 letter code)
countryName_default   = US
stateOrProvinceName   = State or Province Name (full name)
stateOrProvinceName_default = Illinois
localityName          = Locality Name (eg, city)
localityName_default  = Chicago
organizationName          = Organization Name (eg, company)
organizationName_default  = Example, Co.
commonName            = Common Name (eg, YOUR name)
commonName_max        = 64

[ req_ext ]
subjectAltName          = @alt_names

[alt_names]
DNS.1   = test.domain.com
DNS.2   = other.domain.com
DNS.3   = www.domain.net
```

You will need to set your alt\_names section to the FQDNs you wish to use.  If you need more simply add "DNS.4 = whatever.com" and so on.  Once you have done that, save the file as "req.conf" and then execute openssl!

```bash
openssl req -new -nodes -out myreq.csr -config req.conf
```


```
Generating a 1024 bit RSA private key
............................................................++++++
..++++++
writing new private key to 'privkey.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [US]:US
State or Province Name (full name) [Illinois]:Illinois
Locality Name (eg, city) [Chicago]:Chicago
Organization Name (eg, company) [Example, Co.]:Example, Co.
Common Name (eg, YOUR name) []:www.domain.com
```

You now have a "myreq.csr" and a "privkey.pem" associated with the CSR.  You can now submit this CSR to a CA for signing or sign it with your own self-signed CA.  A tutorial to perform the latter will be published in a few days!
