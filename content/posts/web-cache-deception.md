---
title: "Web Cache Deception: Some Real Cases"
date: 2023-12-29
# weight: 1
# aliases: ["/first"]
tags: ["web", "WCD", "bug-bounty", "en"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "real scenarios that I reported in 2023 in bug bounty programs"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

## Web Cache Deception (WCD)
Web Cache Deception (WCD) is an attack in which an attacker tricks a cache proxy into improperly storing private information sent over the Internet, gaining unauthorized access to these cached data. It was proposed by Omer Gil, a security researcher, in 2017.

## Introduction

Many are already aware of what Web Cache Deception is and how its exploitation works, so today I wanted to bring something different and show some real scenarios that I reported in 2023 in bug bounty programs.

### 1. Search Feature

Our first case concerns a web application, most likely Laravel, where it was possible to exploit WCD by abusing the text search feature of the API. The PATH was something like **/v2/search/{WORD}** where {WORD} would be our search term. Changing the PATH to **/v2/search/anything;.css** led the server to respond to our request with the **MISS** Header, and subsequent requests with the **HIT** Header.

Below is an example of the response:

```
HTTP/2 200 OK
Date: Sun, 10 Sep 2023 05:55:41 GMT
Content-Type: application/json
Vary: Accept-Encoding
Cache-Control: max-age=315360000
Set-Cookie: XSRF-TOKEN={COOKIE1}; expires=Tue, 10-Oct-2023 05:55:39 GMT; Max-Age=2592000; path=/; secure
Set-Cookie: toolzz_session={COOKIE2}; expires=Tue, 10-Oct-2023 05:55:39 GMT; Max-Age=2592000; path=/; secure; httponly
Strict-Transport-Security: max-age=15724800; includeSubDomains
Server: EDITED
Expires: Thu, 31 Dec 2037 23:55:55 GMT
Cdn-Location: 
EDITED-Cdn-Cachestatus: HIT
X-Xss-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
Content-Security-Policy: upgrade-insecure-requests

[]
```

In this scenario, we just had to send **https://edited.com/v2/search/VICTIM;.css** to a user, and their cookie would be saved in the page's response. We achieved a high impact with this vulnerability.


### 2. Ignoring Content

This second case was quite similar to the first, but here the exploitation occurred because the application ignored everything that came after ";" in a specific version of the API. This application used the route /v4/me to obtain authenticated user data, but it still had older versions in production (such as v2 and v3) and in v2 specifically, everything after ";" was ignored. Therefore, we could exploit this application by sending **https://edited.com/v2/me;anything.css?params** to a user, and after clicking, their data, which were returned in the page response, would be stored in the cache.

![IMG1](https://cdn-images-1.medium.com/max/800/1*y8g2zslj6uC0ENHN_uxNQg.png)


![IMG1](https://cdn-images-1.medium.com/max/800/1*hyaCCBE9E_9LW4dxyGiEfA.png)


### 3. Abusing Other Functionalities, no ATO


In this scenario, I wanted to bring something different from account theft to show other horizons of exploitation. In this application, we also managed to make a part of the passed path be ignored by the application. While testing some scenarios, I noticed that the application returned a 404 and the path I tried to access, and when using some special characters in URL Encoding, it returned the output decoding. With this, when trying to access API paths passing **%0a-anytext** (\n in URL Encoding), it returned a 200 status code. So, it was just a matter of changing to **/api/user%0a-anytext.css** for us to have our response stored in cache.

![IMG1](https://cdn-images-1.medium.com/max/800/1*PSZ4AMaLutHmgwAq7oNIVg.png)


However, unlike the previous applications, we couldn't exploit an Account Takeover (ATO) in a straightforward manner, as we were not dealing with cookies that were passed and returned in the same request simply. The application used Authorization Bearer, where the token is usually stored in local storage. Therefore, we couldn't just send an API link to a user and have their data saved in the cache, because it's the front-end of the application that constructs the requests for the API and adds the authorization header. As a result, our report was downgraded to a medium impact.



![IMG1](https://cdn-images-1.medium.com/max/800/1*jKFfVY7fB-gM1oR5Wh2sMQ.png)



Focusing on high and critical impact reports, I moved on to another exploitation scenario where I attempted to use WCD to exploit features of the application. One such feature was the invitation to rooms where a user can create an invitation to their room/group and then destroy/deactivate it, making it no longer possible to enter through that invitation.

To exploit this feature, we first created an invitation and used WCD to save the content of this invitation in the cache **/api/invite/INVITE%0a-randomtext.css**. With its content now cached, we can destroy this invitation, but we still have the cached version, allowing us to access the group from a destroyed invitation.

Destroyed invitation, response not found.
![IMG1](https://cdn-images-1.medium.com/max/800/1*tnLvJqBa-kckni_w6obD8g.png)

Destroyed Invitation, Response Found
![IMG1](https://cdn-images-1.medium.com/max/800/1*GgTNEKqolWuJs_sO-6JDfw.png)


After sending it to the program's team, we got our result.

![IMG1](https://cdn-images-1.medium.com/max/800/1*SYfefGNShBbYiwqcfiQVyQ.png)


