---
author: admin
comments: true
layout: post
slug: find-a-matching-certificate-and-key-pair
title: Find A Matching Certificate And Key Pair
wordpress\_id: 806
tags:
- bash
- openssl
---

If you have a list of keys and SSL certs and don't know which cert belongs with which key, here's a script for you.  It's not efficient (nested for loop!), but it gets the job done quickly.[^1]

```bash

#!/bin/bash
for i in `ls *.key`
do
key_mod=`openssl rsa -noout -in $i -modulus`
for j in `ls *.cer`
do
x509_mod=`openssl x509 -noout -in $j -modulus`
if [ "$x509_mod" == "$key_mod" ]; then
echo "$j matches $i"
fi
done
done
```

[^1]: If bash allowed multidimensional or associative arrays this would be trivial to optimize.
