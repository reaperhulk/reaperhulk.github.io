---
author: admin
comments: true
date: 2012-11-02 16:47:58+00:00
layout: post
slug: building-a-ca-r509-howto
title: Building a CA (r509 Howto)
wordpress\_id: 1756
tags:
- ca
- certificate
- crl
- ocsp
- openssl
- ruby
---

**This post has been updated to be compatible with the r509 0.10 API and configuration file format.**

I've covered how to [create a self-signed certificate authority](/2009/01/18/openssl-self-signed-ca/) using OpenSSL in the past, but today we're going to go step by step through the process of building a fully functional CA using [r509](http://r509.org) and its ancillary projects. At the end you'll have a CA capable of issuing and revoking [X.509v3](http://www.ietf.org/rfc/rfc5280.txt) certificates via an HTTP interface[^1], publishing CRLs, and responding to OCSP requests.



### Why? What's this for?


Certificate authorities are useful in any number of environments where you want to establish the identity of a server (or client). Most commonly this takes the form of TLS handshakes on port 443 that are then used to transmit HTTP traffic (HTTPS). This howto aims to help you set up a basic certificate authority that you can use to issue certificates to consumers in your own environment for whatever specific use case you have.



### Are there any caveats?


Of course! For brevity[^2] and simplicity this document does several things that are **not best practice** for running your own CA. We will create a self-signed root, but won't create a subordinate root to issue certs off of. We will use the root itself to sign OCSP responses rather than creating an OCSP signing delegate. We won't talk at all about security around your root certificate private key[^3].



### Setting Up Your Environment


r509 is a ruby project, so you'll want to have ruby. At this time I'd recommend the latest ruby (2.0.0), but r509 is tested against 1.9.3, 2.0.0, 2.1.0, ruby-head, and rubinius. JRuby is not supported because r509 uses the OpenSSL C bindings, which are unavailable there.



### Getting the Gems


Once you have a functional ruby install, you'll need to install some gems. We'll start with r509 (which is available via rubygems.org).
`gem install r509`

Next, we'll need [r509-ca-http](https://github.com/r509/r509-ca-http), a RESTful interface for issuing certificates.
`gem install r509-ca-http`

We'll also want [r509-middleware-validity](https://github.com/r509/r509-middleware-validity) to write validity data for our issued/revoked certs to a data store (in this case, redis). This gem is optional, but without it you will not be able to serve OCSP responses.
`gem install r509-middleware-validity`

Finally, [r509-middleware-certwriter](https://github.com/r509/r509-middleware-certwriter) will write issued certificates to the filesystem for archival purposes.  This gem is **optional**. You may choose to store your issued certificates in whatever way you wish.
`gem install r509-middleware-certwriter`



### Setting Up The Data Store


If we're going to write to [redis](http://redis.io), we'll need it installed. You can [download the source](http://redis.io/download), grab it from your Linux distribution's package manager, or use a tool like [homebrew](https://github.com/mxcl/homebrew) if you're on OS X (just type `brew install redis`). Once installed go ahead and start it up (either directly via `redis-server` or init.d) and leave it running. You can obviously skip this step if you've chosen to not use r509-middleware-validity.



### Create a Root Certificate


Now that we have the gems (and their dependencies) installed we can set up a system to issue certificates. First let's create a new directory called `r509-howto-ca` and cd into it (we'll be working within this directory from now on). r509 comes with a utility (cleverly called "r509"). Simply type `r509 --out root.cer --keyout root.key` in the shell and it will enter an interactive mode that will let us generate the root certificate quickly and easily. You'll need to choose the following:




  * CSR Bit Length - This defaults to 2048. You can hit enter or type a different # here (like 4096) if you'd prefer


  * Message Digest - Defaults to sha1. Hit enter or type another digest (e.g. sha256)


  * C - This stands for country. Use a two letter ISO country code.


  * ST - This is state or province. You can type whatever you'd like here.


  * L - This is locality (city/town). You can type whatever you'd like here.


  * O - This is organization. Again, whatever you'd like.


  * OU - This is organizational unit. This is frequently left blank in certificates, but you can optionally enter something here (like "IT" for example)


  * CN - This is the common name. In the case of root certificates it is essentially the "name" of the certificate. You can choose whatever you'd like, but I recommend something like "MyCompany Root CA"


  * SAN Domains - Leave this blank and hit enter


  * Self-signed cert duration in days - use 3650 (or higher if you'd like greater than 10 year validity)


After these steps the script will write a certificate and private key to the files specified.



### Set up the yaml configuration


r509 is largely configured through [yaml](http://yaml.org) files. Here's an example configuration we're going to customize for your needs.

```yaml

---
certificate_authorities:
  r509_howto_ca:
    ca_cert:
      cert: root.cer
      key: root.key
    ocsp_start_skew_seconds: 3600
    ocsp_validity_hours: 168
    crl_list_file: r509_howto_ca_list.txt
    crl_number_file: r509_howto_ca_crlnumber.txt
    crl_validity_hours: 168
    crl_md: SHA1
    profiles:
      server:
        basic_constraints:
          :ca: false
        authority_info_access:
          :ocsp_location:
          -  :type: URI
             :value: http://ocsp.myca.com:9291
        crl_distribution_points:
          :value:
          -  :type: URI
             :value: http://crl.domain.com/r509_howto_ca.crl
        key_usage:
          :value:
          - digitalSignature
          - keyEncipherment
        extended_key_usage:
          :value:
          - serverAuth
        default_md: SHA1
        allowed_mds:
        - SHA256
        - SHA512
        - SHA1
        subject_item_policy:
          CN:
            :policy: required
          O:
            :policy: required
          OU:
            :policy: optional
          ST:
            :policy: required
          C:
            :policy: required
          L:
            :policy: required
certwriter:
  path: /absolute/path/to/wherever/you/want/the/certs

```


Phew, there's a lot in there (and this is a simple example!). You can check out the [README](http://r509.org/doc/index.html) to learn more (including how to build configs programmatically), but for now copy this config exactly and write it to a file named `config.yaml` in the same directory as your root+key.

**r509-middleware-certwriter:** Skip this if you are not using this gem. If you are, update the path parameter at the bottom to point to the absolute path where you want it to write certificates. This directory must exist.

**r509-middleware-validity:** Skip this if you are not using this gem. If you are, start up redis using the command `redis-server` or via init.d. You'll want to leave this running for the remainder of the tutorial.

You'll also need to create empty files named `r509_howto_ca_crlnumber.txt` and `r509_howto_ca_list.txt` in the same directory. You can do this via `touch r509_howto_ca_crlnumber.txt r509_howto_ca_list.txt`.



### Adding the r509-ca-http rackup file


r509-ca-http is a gem that contains a [sinatra](http://sinatrarb.com) server for performing simple certificate issuance. To start up the server you'll need a rackup file. Write the [config.ru](https://github.com/r509/r509-ca-http/blob/master/config.ru) file to disk in the same directory as everything else.


### Running the Server


We've got everything in place, so just type `rackup -p 9292`, hit enter, and you should see output something like this (although your details will vary):

```

I, [2013-08-31T22:41:51.593474 #73412]  INFO -- : Using r509 middleware certwriter
I, [2013-08-31T22:41:51.645133 #73412]  INFO -- : Config:
I, [2013-08-31T22:41:51.645234 #73412]  INFO -- : CA Cert:/C=US/ST=Illinois/L=Chicago/O=r509 LLC/CN=r509 CA Tutorial
I, [2013-08-31T22:41:51.645278 #73412]  INFO -- : OCSP Cert (may be the same as above):/C=US/ST=Illinois/L=Chicago/O=r509 LLC/CN=r509 CA Tutorial
I, [2013-08-31T22:41:51.645301 #73412]  INFO -- : OCSP Validity Hours: 168
I, [2013-08-31T22:41:51.645319 #73412]  INFO -- : CRL Validity Hours: 168
I, [2013-08-31T22:41:51.645336 #73412]  INFO -- :

[2013-08-31 22:41:51] INFO  WEBrick 1.3.1
[2013-08-31 22:41:51] INFO  ruby 2.0.0 (2013-05-14) [x86_64-darwin12.4.0]
[2013-08-31 22:41:51] INFO  WEBrick::HTTPServer#start: pid=73412 port=9292

```


The server is now up and running on port 9292! The following test interfaces are available:




  * http://localhost:9292/test/certificate/issue


  * http://localhost:9292/test/certificate/revoke


  * http://localhost:9292/test/certificate/unrevoke


These interfaces are meant solely for testing. In a production environment please look at the [r509-ca-http API](https://github.com/r509/r509-ca-http#api).

Since we're testing let's go ahead and open a web browser and go to http://localhost:9292/test/certificate/issue

To issue a certificate off our root we're going to need to fill in quite a bit of data. For security reasons r509-ca-http overwrites the data[^4] from a certificate signing request (CSR) with the data you specify. Let's fill out the fields:




  * CA - enter `r509_howto_ca`[^5]


  * Profile - enter `server`[^6]


  * Validity Period - enter `31536000` (this is 1 year in seconds)


  * Fill out, C, ST, L, O, and CN with the data you want.


  * Leave SAN domains and emailAddress blank.


  * Paste a CSR in the CSR textarea. You can quickly generate a CSR by using the `r509` script again, but be sure you leave "Self-signed cert duration in days" blank. You must include the -----BEGIN CERTIFICATE REQUEST----- and -----END CERTIFICATE REQUEST----- lines.


Now click "gimme" and you'll get your certificate! You should copy the cert and write it to a file named `cert.pem`.

Note: If you're using the r509-middleware-validity gem or r509-middleware-certwriter you will also have a record of this issuance in redis or the cert will be written to the filesystem for archival purposes.



### Revocation


We've issued a certificate now, but what if we want to revoke it? Head to http://localhost:9292/test/certificate/revoke

There are 3 fields on this page. CA, serial, and reason.




  * Enter `r509_howto_ca` for CA


  * For serial, you'll want the serial number of your previously issued certificate. Run `r509-parse cert.pem` to get some details about your cert, then copy the value marked "Serial (Decimal)" and paste it into the serial field


  * Ignore the reason field for now.[^7]


Now hit revoke and you'll be given a PEM encoded X.509 CRL! You can also look in the r509\_howto\_ca\_list.txt we created earlier to see that the serial is listed as revoked.



### CRL Generation


So we have a certificate and we revoked it, but how would a client who reaches a server using the certificate know that it was revoked? SSL certificates offer two methods for providing revocation information to relying parties. Let's talk about Certificate Revocation Lists (CRLs) first.

Whenever a client receives a certificate it performs numerous checks[^8] to ascertain the acceptability of the certificate. One of those is looking within the certificate for a CRL Distribution Point (CDP). If found, the client fetches the CRL from the URL provided, validates it, and checks to see if the certificate in question is listed in the CRL. In our config.yaml above we had a line for crl\_distribution\_points: `http://crl.domain.com/r509_howto_ca.crl`. If you want your certificates to be able to check CRL you'll need to change this to a domain name that you can control. This is not necessary for this tutorial (we'll be using OCSP so read on!).

Once you've made that change you'll need to set up something to copy CRLs obtained from `http://localhost:9292/1/crl/r509_howto_ca/generate` to a web accessible area[^9].



### OCSP Responses


CRLs are nice to have, but the more modern solution for certificate revocation information is the Online Certificate Status Protocol (OCSP) (RFC [2560](http://www.ietf.org/rfc/rfc2560.txt) and [5019](http://www.ietf.org/rfc/rfc5019.txt)). To use r509's OCSP responder capabilities you will need to be using r509-middleware-validity, as well as installing one more gem. Time to `gem install r509-ocsp-responder`.

We are going to use the root to sign responses for our OCSP responder. This is not considered best practice for a variety of reasons, but it will simplify our setup. We'll cover OCSP signing delegates in another post. For now, let's set up the responder's config yaml. The text below should be placed in a file named config.yaml, but not in the same directory as your CA. Let's create an `ocsp` directory and place it in there. You'll also want to copy the root cert and key from the r509\_howto\_ca directory into the ocsp directory (or you can change the paths below).

```yaml

---
copy_nonce: true
cache_headers: true
max_cache_age: 60
certificate_authorities:
  r509_howto_ca:
    ca_cert:
      cert: root.cer
      key: root.key

```

The first 3 params are described in the [responder documentation](https://github.com/r509/r509-ocsp-responder#options) and the certificate\_authorities node is just the single CA we want to respond for at this time.

Next we'll need the [rackup file](https://github.com/r509/r509-ocsp-responder/blob/master/config.ru) (written to config.ru just like last time, but in the same `ocsp` directory as the new config.yaml you just created).

If you want higher performance lookups you can optionally install the `hiredis` gem.

We can now start up the responder! `rackup -p 9291`

```

W, [2013-09-01T10:11:01.371540 #77741]  WARN -- : Loading redis with standard ruby driver
W, [2013-09-01T10:11:01.376102 #77741]  WARN -- : Config loaded
W, [2013-09-01T10:11:01.376190 #77741]  WARN -- : Copy Nonce: true
W, [2013-09-01T10:11:01.376224 #77741]  WARN -- : Cache Headers: true
W, [2013-09-01T10:11:01.376252 #77741]  WARN -- : Max Cache Age: 60
W, [2013-09-01T10:11:01.376294 #77741]  WARN -- : Config:
W, [2013-09-01T10:11:01.376382 #77741]  WARN -- : CA Cert:/C=US/ST=Illinois/L=Chicago/O=r509 LLC/CN=r509 CA Tutorial
W, [2013-09-01T10:11:01.376463 #77741]  WARN -- : OCSP Cert (may be the same as above):/C=US/ST=Illinois/L=Chicago/O=r509 LLC/CN=r509 CA Tutorial
W, [2013-09-01T10:11:01.376498 #77741]  WARN -- : OCSP Validity Hours: 168
W, [2013-09-01T10:11:01.376515 #77741]  WARN -- :

[2013-09-01 10:11:01] INFO  WEBrick 1.3.1
[2013-09-01 10:11:01] INFO  ruby 2.0.0 (2013-05-14) [x86_64-darwin12.4.0]
[2013-09-01 10:11:01] INFO  WEBrick::HTTPServer#start: pid=77741 port=9291

```


Awesome! If we were running a full CA here we'd want to update our r509-ca-http config yaml to have the correct OCSP URI, but the config from earlier is perfect for testing. Speaking of testing...let's test this thing out!



### Testing OCSP Responses


We need to issue a new certificate using the test interface. First create a new directory called `test\_server` and cd into it, then do the following:




  * Create a new csr and key using `r509 --keyout localhost.key --subject "/CN=localhost/O=r509 test"`


  * Head to http://localhost:9292/test/certificate/issue and issue the certificate, but **be sure you set the CN to localhost**


  * Save the resulting certificate to a file named `localhost.cer`



You should now have two files (localhost.cer and localhost.key) in your test\_server directory. Inside that directory create a third file named `server.rb` with the following data in it:

```ruby

require 'webrick/https'
require 'r509'

cert = R509::Cert.load_from_file('localhost.cer').cert
key = R509::PrivateKey.load_from_file('localhost.key').key

s = WEBrick::HTTPServer.new(
 :Port => 8443,
 :DocumentRoot => Dir::pwd,
 :SSLEnable => true,
 :SSLCertificate => cert,
 :SSLPrivateKey => key
)
trap("INT"){ s.shutdown }
s.start

```


You should also add this line to your /etc/hosts file:
`127.0.0.1    ocsp.myca.com`

Finally, we need to set up a web browser to trust our new root CA. Let's use Firefox[^10]. Open the preferences, click the advanced tab, then encryption, then click the view certificates button. You now have a list of certificates in various categories. Click the authorities tab at the top, then click "Import..." and choose the root CA we created (`root.cer`). Do **not** select localhost.cer. Mark the certificate as trusted for websites and then exit out of the preferences.

We're now ready to test! Make sure your redis server is running (it should have been running when you issued the certificate! If not, issue another one) along with r509-ocsp-responder (`rackup -p 9291` in the ocsp directory), then type `ruby server.rb` inside the `test\_server` directory we just created above. Now open Firefox and navigate to https://localhost:8443 . If all goes well you should see it load the page with no problems and some log lines indicating the OCSP request occurred should show up in the ocsp responder output! Congratulations, you now have a functioning CA. Feel free to revoke the certificate and see Firefox reject you!



### Quick Start


You can also [download a zip](/assets/media/2012/11/r509-howto-ca-0.10.zip) file of the configurations and directory setup required to get started. Grab it and then check out the README.txt to get started. You'll still need to generate your own certs, but this will give you all the config.ru and config.yaml files already in place.



### Github Project Pages


r509 consists of quite a few gems. Here are all their GitHub project pages. Fork, contribute, complain (maybe not that last one)!
[r509](https://github.com/r509/r509)
[r509-ocsp-responder](https://github.com/r509/r509-ocsp-responder)
[r509-ca-http](https://github.com/r509/r509-ca-http)
[r509-middleware-validity](https://github.com/r509/r509-middleware-validity)
[r509-middleware-certwriter](https://github.com/r509/r509-middleware-certwriter)
[r509-validity-redis](https://github.com/r509/r509-validity-redis)
[r509-validity-crl](https://github.com/r509/r509-validity-crl) (A simple project to make the OCSP responder consume a CRL. Not recommended, but possible)

[^1]: You'll want to secure this for any serious use obviously.

[^2]: Relative brevity. If you're down here in the footnotes you know this is not a brief howto.

[^3]: This is an article unto itself. Suffice it to say, you **must** protect that key if you intend to use your CA for any serious purpose. The compromise of that key to third parties renders your entire CA hierarchy useless for identity purposes. Public CAs use [HSMs](http://en.wikipedia.org/wiki/Hardware_security_module) and various multi-factor physical access controls to limit the risk of this occurring.

[^4]: With the obvious exception of the public key

[^5]: This is the name of the config of the certificate authority we wish to issue a certificate from. You can see it in the config.yaml file

[^6]: This is the issuance profile we want to use with the CA we've selected. You can have many issuance profiles, each corresponding to a certificate type you want to allow

[^7]: Reason codes are optional. The r509 documentation has a [list of reason codes](http://r509.org/doc/R509/CRL/Administrator.html#revoke_cert-instance_method).

[^8]: Well, they're supposed to at least.

[^9]: In a production environment you'd want this to be a separate server. Your CA should never be directly accessible on the web!

[^10]: We're using Firefox because it checks OCSP by default, is tolerant of OCSP responders that resolve into the RFC 1918 address space, and is relatively easy to add root certificates to
