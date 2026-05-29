---
title: "I Got HyperLink Injection and Breaking a Cryptography Signed OpenRedirect Because One Simple Mistake and Made It Extremely Perfect Because What They Call it By Design"
date: 2026-05-16 00:00:00 +0000
categories: [Web]
tags: [Hyper Link Injection, Open Redirect, Poor Design]
---

Hello Friend, Today I’m Going To Explain How I Found a Hyperlink Injection Vulnerability With Update Email () . It Leaded To Generating Cryptography Signed Trusted Links Without Knowing the Secret Key (Breaking Open Redirect) !

![Jesse Tired of “By Design” Word, But He Will Figure It Out To Get a Valid Bug and Make Them Approve](../assets/img/POST(2)/a1d2afee8c6f974777a013df4b1cae23.jpg)
*Jesse Tired of “By Design” Word, But He Will Figure It Out To Get a Valid Bug and Make Them Approve*

---

### HyperLink Injection :

If You Don’t Know, Hyper Link Injection is A Vulnerability Where There is An Input Will Be Reflected in Automated Template Like a Mail. But, The Attacker Can Exploit the Poor Sanitization and Inject Malicious Links Into That Message. Then, The User Sees It Comes From a Trusted Brand or Company And Clicks It .

![AI Summary](../assets/img/POST(2)/1.png)
*AI Summary*

---

### Chained Informative Bugs :

**It Begin With a Weird Behaviour Which is When I Tried to Edit the Username To Something Like `<h1>Name</h1>` It Passed me Without Any Alert or Something, That’s Not a Bug. But, It Caught Me Attention Of How Can I Exploit This ?!**

![HTML Tags Passed Normally](../assets/img/POST(2)/2.png)
*Html Passed Without Any Problems*

I Thought For a While, and **Recognized** That When I **Update the Email** a ***Mail (Triggered) Sent To The New Email*** Saying The `Settings of The Account Has Updated` *Because “By Design” in The App There is Nothing Called Email Verfication* !

And the Firstname of The Current User Was Sent in The Body of The Email . So, I Got The **IDEA** About Testing For Hyper Link Injection. And If HTML Tags Was Rendered (HTML Injection) .

Updated the Name, Style Color Was This Because The App Was `White` and `rgb(19,68,106)`

```html
<h1 style="color: rgb(19,68,106)">Free Coupon</h1><a href="https://evil.com">Gift</a><br>Anonymous
```

![Before the Attack](../assets/img/POST(2)/5.png)
*Set the Username To a Malicious One*

And What I Expected Happened, We Got Hyperlink Injection Via Updating Email Function

![HyperLink Injection Exists](../assets/img/POST(2)/6.png)
*We Have Got HLI Successfully Via Update Email Function*

And I Got Another Something Interesting, When My Mouse Was Over the <a> Tag Gift . It Appeared Under

```
http://click.<redacted>.com/ss/c/u001.ovViWmuu4Gczm1GLN92JiQk_MXUaimqdAbCwFNYr2Xg/4qn/M4ftzsaZRLCAw3BR7QYzdQ/h5/h001.M2q3JRO1BWqfnhJfGYH1IZmYirXW38svDk40WJK3xm8
```

**Which, I Discovered Later That All Links in The Template Before They Sent to The User They Will Got Likely a Special Valid Encrypted Hash Which Redirects To The Legit Contact, Shop, Other Links …**

**And We Got Another Likely a Very Hard To Get Open Redirect, We Break the Cryptography By Just Triggering the System To Generate What We Want We Don’t Need To Know the Key Just Execute, Which the Developer Was Totally Ensure That No One Will find The Secret Key. But, He Missed That Another Bugs Can Give What The Encrypted Data in the Link Will Redirect To !**

_Which, is A Separate Vulnerability and Made The HyperLink Injection Obfuscated and More Trusted [Perfect]_

![Redirection](../assets/img/POST(2)/7.png)
*Redirection*

![Before the Attack](../assets/img/POST(2)/cr7-cristiano-ronaldo-gif-xjzge0en39h8rijn.gif)
*GoodBye*