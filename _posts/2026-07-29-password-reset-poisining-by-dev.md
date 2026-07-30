---
title: "The Developers Gave Me the Golden Plate to Poison Their Password Reset Links"
date: 2026-07-29 00:00:00 +0000
categories: [Web, Injection]
tags: [Password Reset Poisoning]
image:
  path: ../assets/img/POST(8)/thumb.gif
---

Usually attackers gain password reset poisoning via Host Header Injection like Manipulating the `Host: ` header-value to `attacker.com` or Using Techiniques like `X-Forwarded-Host: attacker.com` but what if the developers made the process very easy and obvious .

## Exploitation : 

> note: the application only send 1 Password Reset OTP, that mean when attacker exploit this the victim must click the link to request a new one even after 1 year. that made the vuln more powerful !
    ![](../assets/img/POST(8)/blocked.png)

okay, after We created a test account let's go back and request a password-reset link

![](../assets/img/POST(8)/1.png)
*interface*

Going to burp and Saw that the request was that the **{token}** value is _inserted from_ the **client-side**

![](../assets/img/POST(8)/2.png)
*api*

so, I Changed the Legit site link to Webhook Unique url to get the exploitation poc . in the first I Doubted the exploit would work because it's too easy and the server will reject the injected domain and give me 403 forbidden for example but **it was not**

![](../assets/img/POST(8)/3.png)
*It Passed Normally*

Going the Email Inbox and Sees That It Worked without Stripping the domain 

![](../assets/img/POST(8)/4.png)
*Email Inbox*

### Victim Side :

requesting the sent link to be able to request a new one, or just he will click it for any other reason

![](../assets/img/POST(8)/request.png)
*requesting*

the Attacker Will Go Back to his Maliciuos Domain, In my case: webhook.site

![](../assets/img/POST(8)/5.png)
*Exploited*

requesting the token with the target domain enabling me to change the password for that user and get into his account

![](../assets/img/POST(8)/6.png)
*now, the attacker can change the password of the victim account*

![](../assets/img/POST(8)/end.gif)
*goodbye*