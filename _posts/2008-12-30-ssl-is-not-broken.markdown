---
author: admin
comments: true
layout: post
slug: ssl-is-not-broken
title: SSL Is Not Broken
wordpress\_id: 6
tags:
- ssl
---

This morning an important and ingenious [method](http://www.win.tue.nl/hashclash/rogue-ca/) of compromising the chain of trust for PKI was published.  Naturally the internet is in a tizzy about the implications of this break, but misinformation rules the day.

SSL is in no way broken, but CAs still issuing MD5 certificates simply **MUST** stop as soon as possible. There are really two components to this (extremely clever) break:

1) a further reduction in the time required to compute a useful MD5 collision which the authors of the paper have not yet detailed.

2) CAs not using randomized serials to increase the entropy provided to the hash function.

If #2 is immediately rectified by the CAs then the difficulty of this attack increases dramatically. Of course, if they have any sanity they will switch to both SHA-1 and randomized serials.

It's important to note, for those who don't have the patience to read through the paper, that this describes a method of issuing a new subordinate CA from[^1] a trusted CA that can, in turn, issue any certificate without proper vetting. These certs would be trusted by every browser[^2] and OS containing the cert used for the initial issuance. This is not a compromise of MD5 certificates in the wild.

That distinction may seem inconsequential in view of the potential for a completely broken chain of trust, but it is important. Provided that the potentially vulnerable[^3] CAs (RapidSSL, possibly others) remedy their issues then this attack is completely neutered until such time as SHA1 is compromised to a similar extent. If those CAs take quick action then drastic steps (such as the revocation of whole root CAs and mass reissuance to all affected customers) should not be necessary.

 

**Update (1/11/09): **For future reference, RapidSSL ceased using MD5 signatures in their certificates within a day of the release of this paper.

[^1]: The actual subordinate CA is not being signed by the CA, only a valid cert, so saying it's "from" the CA is misleading.

[^2]: With the exception of Opera, which requires a valid CRL/OCSP response from every link in the chain. As described in the paper the authors were unable to include a crlDistributionPoint node in their ASN.1 structure so while browsers such as IE, Firefox, and Safari would simply trust the cert, Opera would explicitly deny trust.

[^3]: Being vulnerable to this attack requires both the active use of MD5 in new certificates, an exactly guessable validity via automated issuance or other means, and sequential or otherwise predictable serial numbers.
