---
title: DCV Delegation for Partial Zones
date: 2023-09-21 12:00:00 -500
categories: [Cloudflare, DCV]
tags: [cname, dns]     # TAG names should always be lowercase
---

## DNS > Record (CNAME Setup)

![]({{ site.baseurl }}/assets/images/CNameDCV/1.CNAMEDNS.png)


If you want to host a website name ```abc.jjonflare.cloud``` you need to create a partial Zone suffix in your authoritative DNS provider.

![]({{ site.baseurl }}/assets/images/CNameDCV/2.partialdns.png)

It will look like ```abc.jjonflare.cloud.cdn.cloudflare.net```

**Example on Godaddy DNS**


![]({{ site.baseurl }}/assets/images/CNameDCV/3.GodaddyDNS.png)


Next create CNAME record in your DNS provider for DCV delegation



**Example on Godaddy DNS**

_acme-challenge.abc abc.jjonflare.cloud.c43c00fXXXXXXX.dcv.cloudflare.com.

![]({{ site.baseurl }}/assets/images/CNameDCV/4.DCV.png)

**On Cloudflare Dashboard**

Next add the CNAME record on Cloudflare dashboard ```abc.jjonflare.cloud```

![]({{ site.baseurl }}/assets/images/CNameDCV/5.CloudflareDNS.png)

**Edge Certificate**

Navigate to SSL/TLS > Edge Certificate, order advanced certificate
For this example I selected **Google Trust Services** as Cloudflare will stop using **DigiCert** as an issuing certificate authority (CA) [More info here](https://developers.cloudflare.com/ssl/reference/migration-guides/digicert-update/advanced-certificates/)


![]({{ site.baseurl }}/assets/images/CNameDCV/6.OrderAdvcert.png)
![]({{ site.baseurl }}/assets/images/CNameDCV/7.googletrust.png)

And finally the certificate is issued on Cloudflare
![]({{ site.baseurl }}/assets/images/CNameDCV/8.Certissued.png)
