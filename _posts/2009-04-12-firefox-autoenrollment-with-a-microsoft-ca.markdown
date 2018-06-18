---
author: admin
comments: true
layout: post
slug: firefox-autoenrollment-with-a-microsoft-ca
title: Firefox Autoenrollment With A Microsoft CA
wordpress\_id: 471
tags:
- crypto
- ssl
---

If you're running a Microsoft CA and you want to be able to accept enrollment requests from clients supporting keygen (Firefox, Safari, Opera, et cetera) you've probably found that the /certsrv/ page allows enrollment, but the requests fail when you attempt to issue the certificate.  This is because the server is not parsing the subject attributes from the request.  To fix this, run the following on your server as administrator on the command line.

```
certutil -setreg ca\CRLFlags +CRLF_ALLOW_REQUEST_ATTRIBUTE_SUBJECT
```

You can also set your server to auto-issue on request for certain certificate profiles.  To do this add the CA snap-in and get properties of your CA.  Under the policy module tab click properties again and click the "Follow the settings.." radio button.
[![add-snapin](/assets/media/2009/03/add-snapin-150x150.png)](/assets/media/2009/03/add-snapin.png)[![mmc](/assets/media/2009/04/mmc-150x150.png)](/assets/media/2009/04/mmc.png)

[](/assets/media/2009/04/mmc.png)[![properties](/assets/media/2009/03/properties-150x150.png)](/assets/media/2009/03/properties.png)[![requesthandling](/assets/media/2009/03/requesthandling-150x150.png)](/assets/media/2009/03/requesthandling.png)
