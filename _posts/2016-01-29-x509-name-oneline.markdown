---
layout: post
title: Anatomy of a MySQL CVE&#58; X509_NAME_oneline Certificate Validation
slug: x509-name-oneline
---

**Update:** MySQL 5.6.30 was released on 2016/4/11.

[CVE-2016-2047](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2047) was recently [disclosed by MariaDB](http://www.openwall.com/lists/oss-security/2016/01/26/3), so despite the fact that no fix is yet available for MySQL here's a quick rundown of what the vulnerability is.

**Summary:** A man-in-the-middle attacker who can obtain a trusted TLS certificate with a specially crafted subject name can trick a MySQL client into trusting a malicious server.

**Affected:** Possibly every version MySQL (and by extension Percona, MariaDB, etc) from 4.0 and on. The bug was originally introduced by this [commit](https://github.com/mysql/mysql-server/commit/a51668c74c498e3e5702342fe5ced68afbee0756#diff-5b45fa673c88c06a9651c7906364f592R1590) on April 18, 2006.

When a MySQL client connects to a TLS enabled server it needs to verify the certificate[^7] if `MYSQL_OPT_SSL_VERIFY_SERVER_CERT` is set. The verification code looks like this:

```cpp
static int ssl_verify_server_cert(Vio *vio, const char* server_hostname, const char **errptr)
{
  SSL *ssl;
  X509 *server_cert;
  char *cp1, *cp2;
  char buf[256];
  DBUG_ENTER("ssl_verify_server_cert");
  DBUG_PRINT("enter", ("server_hostname: %s", server_hostname));

  if (!(ssl= (SSL*)vio->ssl_arg))
  {
    *errptr= "No SSL pointer found";
    DBUG_RETURN(1);
  }

  if (!server_hostname)
  {
    *errptr= "No server hostname supplied";
    DBUG_RETURN(1);
  }

  if (!(server_cert= SSL_get_peer_certificate(ssl)))
  {
    *errptr= "Could not get server certificate";
    DBUG_RETURN(1);
  }

  if (X509_V_OK != SSL_get_verify_result(ssl))
  {
    *errptr= "Failed to verify the server certificate";
    X509_free(server_cert);
    DBUG_RETURN(1);
  }
  /*
    We already know that the certificate exchanged was valid; the SSL library
    handled that. Now we need to verify that the contents of the certificate
    are what we expect.
  */

  X509_NAME_oneline(X509_get_subject_name(server_cert), buf, sizeof(buf));
  X509_free (server_cert);

  DBUG_PRINT("info", ("hostname in cert: %s", buf));
  cp1= strstr(buf, "/CN=");
  if (cp1)
  {
    cp1+= 4; /* Skip the "/CN=" that we found */
    /* Search for next / which might be the delimiter for email */
    cp2= strchr(cp1, '/');
    if (cp2)
      *cp2= '\0';
    DBUG_PRINT("info", ("Server hostname in cert: %s", cp1));
    if (!strcmp(cp1, server_hostname))
    {
      /* Success */
      DBUG_RETURN(0);
    }
  }
  *errptr= "SSL certificate validation failure";
  DBUG_RETURN(1);
}
```

`SSL_get_verify_result` is a function from OpenSSL that will (ideally) validate
the following:

* A chain can be built to a trusted root (the set of which were previously supplied).
* Each signature in the chain is valid.
* Each certificate in the chain is within the validity window.
* Each certificate in the chain is not revoked[^1].
* Each certificate is valid for the purpose it is being used (e.g. subordinate certificates have `CA:TRUE` values in `basicConstraints`, `nameConstraints` and `extendedKeyUsage` rules are enforced[^2], etc)

What it does **not** do is verify that the certificate supplied is the one you expected to see. This is typically done by having the client compare the domain it attempted to connect to (`mysql.myunhackablestartup.net`) to fields inside the certificate. In the long long ago, in the days before [RFC 2818](https://tools.ietf.org/html/rfc2818) this was done with the `commonName` (aka `CN`) value inside the subject. This value is still available, but in the past few years browsers and other client software have finally transitioned to depending on values with an X.509 extension called [Subject Alternative Name](https://tools.ietf.org/html/rfc5280#section-4.2.1.6). Within a `subjectAltName` you will typically find one or more `dNSName` entries[^3]. Those must be verified against what the client requested using [RFC 6125](https://tools.ietf.org/html/rfc6125#section-6.4) rules.

Okay, so that's a rough[^4] description of how name validation should work. To accomplish this task the MySQL client does the following:

* Creates a fixed length character buffer (256 bytes).
* `X509_NAME_oneline` serializes an `X509_NAME` into the previously mentioned character buffer. It does this by slash delimiting each `X509_NAME_ENTRY`.
* Scans for `/CN=` via `strstr` to find the `commonName`.
* Does pointer arithmetic to skip 4 bytes, which corresponds to `/CN=`.
* Searches for a `/` via `strchr`. If found, it dereferences the returned `char *` and writes a null byte.
* Uses `strcmp` to compare `cp1` to the supplied server hostname.

This is problematic for a number of reasons. Starting from the beginning:

* A fixed length buffer makes no sense here as a subject line could easily be longer than that. As is, anything longer than 256 bytes gets implicitly truncated. And, since several of the later functions assume a null-terminated string we're into some unpleasant territory.
* Creating a null-terminated string by inserting a null byte using `strchr` so you can perform a `strcmp` is not really a great idea.
* `strcmp` is not exactly RFC 6125[^5].

And of course, the big one: `X509_NAME_oneline` serializes the `X509_NAME` ASN.1 structure, but it is important to note that there is **nothing special about the slash character it uses as a delimiter**. It can be encoded into a name entry and is thus not a unique character that can unambiguously separate each entry.

### Exploiting

To exploit this you need to get a CA[^6] trusted by the MySQL client you want to attack to issue a certificate where you set a subject entry to `/CN=mysql.myunhackablestartup.net`. This can be any name entry, but it needs to be encoded **before** the "real" CN. For example, if you `X509_NAME` has a structure like this:

```ruby
[
	['O', 'myorg/CN=mysql.myunhackablestartup.net'],
	['C', 'US'],
	['CN', 'someotherdomain.local']
]
```

Then the `X509_NAME_oneline` serialization of this would look like `/O=myorg/CN=mysql.myunhackablestartup.net/C=US/CN=someotherdomain.local` and would pass validation with the MySQL client despite having an entirely different `commonName`.

### Fixed Versions

MariaDB has already released fixes for this as 5.5.47, 10.0.23, and 10.1.10. Oracle and Percona have not yet released patches, but I will update this post when they do. Please upgrade if you're relying on TLS's security guarantees for your MySQL network communication.


[^1]: Whether or not it does this is a complex topic, but for the purposes of X.509 in this context you can assume revocation is [broken](https://www.imperialviolet.org/2011/03/18/revocation.html).

[^2]: They almost never are so you can't rely on that. Sorry.

[^3]: There are also `iPAddress`, `directoryName`, and several more, but `dNSName` is by far the most common. Take a look at the RFC if you want to see the full list.

[^4]: If you only care about validating `dNSName` then this is a pretty good description, although much like anything in TLS/X.509 land there may be exceptions to work around legacy bugs or existing software. Potentially subtle things like properly handling null bytes [can](http://hackaday.com/2009/07/29/black-hat-2009-breaking-ssl-with-null-characters/) [lead](https://www.ruby-lang.org/en/news/2013/06/27/hostname-check-bypassing-vulnerability-in-openssl-client-cve-2013-4073/) [to](https://bugs.python.org/issue18709) [vulnerabilities](http://tools.cisco.com/security/center/viewAlert.x?alertId=18852).

[^5]: Yes, RFC 6125 was published after this code was originally written, but that RFC is really just a centralized codification of the rules that had evolved over a decade+ of TLS client development.

[^6]: Most public CAs will catch this sort of behavior and refuse to issue such a certificate, but there are hundreds of CAs and it's likely more than a few of them would issue certificates with subjects like this.

[^7]: And validating certificates in non-browser software is [the most dangerous code in the world](https://crypto.stanford.edu/~dabo/pubs/abstracts/ssl-client-bugs.html).
