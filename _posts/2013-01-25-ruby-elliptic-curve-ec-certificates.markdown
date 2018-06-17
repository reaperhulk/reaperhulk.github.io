---
author: admin
comments: true
date: 2013-01-25 19:22:35+00:00
layout: post
slug: ruby-elliptic-curve-ec-certificates
title: Ruby Elliptic Curve (EC) Certificates
wordpress\_id: 1871
---

I've been working on elliptic curve support in a [branch](https://github.com/r509/r509/tree/ec) of [r509](http://r509.org), my Ruby project for managing X.509 certificates, and I'm pleased to say that it's reached the point where it is mostly complete. I'm going to let it bake in the branch for a bit longer and then merge back into master for an eventual v0.9 release. If you need/want to issue EC-based certificates in Ruby then hopefully it will help; it papers over most of the (very serious) issues with the OpenSSL elliptic curve bindings in Ruby with the standard r509 API. [Clone the project](https://github.com/r509/r509/tree/ec), [check out the documentation](http://r509.org/doc/index.html), or just [download](/assets/media/2013/01/r509-0.9ec.gem_.gz) the prebuilt gem.

For those who are using r509 with [r509-ca-http](https://github.com/r509/r509-ca-http) please note that you'll need the current HEAD of the master branch in git to be able to revoke/unrevoke as the ec branch of r509 also contains a significant refactor of the CRL handling code.



### Examples


Here's how you'd create a CSR with an ECDSA private key. (You can also look at the R509::Csr object docs)

```ruby

csr = R509::Csr.new(
  :type => :ec,
  :subject => [
    ['CN','somedomain.com'],
    ['O','My Org'],
    ['L','City'],
    ['ST','State'],
    ['C','US']
  ]
)

```


Specify a specific curve (default is secp384r1):

```ruby

csr = R509::Csr.new(
  :type => :ec,
  :curve_name => 'sect283r1',
  :subject => [
    ['CN','somedomain.com'],
    ['O','My Org'],
    ['L','City'],
    ['ST','State'],
    ['C','US']
  ]
)

```


You can also use the r509 command line script to generate elliptic curve CSRs or self-signed certificates. Type `r509 --help` for instructions or you can invoke it in interactive mode:

```bash
r509 --type ec
```

