---
title: "DeepLink Misconfiguration: Accessing PIX and Account Data"
date: 2024-05-26
# weight: 1
# aliases: ("/first")
tags: ("mobile", "en")
author: "Me"
# author: ("Me", "You") # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Analisando código do malware pixpirate"
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
cover:
    image: "https://cdn-images-1.medium.com/max/800/1*GWZv7nO5EqwGKzWNLJCJ8Q.jpeg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/thalysonz.github.io/posts/dicas-para-análise-de-código-obfuscado"
    Text: "s" # edit text
    appendFilePath: true # to append file path to Edit link
---

While analyzing an application, I noticed that some deeplink calls contained the "path" parameter, and this parameter was being used to define the path of the WebView that the application was accessing.

![IMG](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*wWBYh7-N0u06U4IYV_XyVQ.png)



### Controlling the Path with Deeplinks

So, we could control the path that the WebView was loading, but what if there was an "@" in the path parameter?

If we try to put an "@" in the path parameter, we can control the site that will be loaded in the application's WebView because instead of accessing `https://help-webview.com`, the WebView will load whatever comes after the "@".

![IMG](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*zCwIgHCFhgCyZrNTB93WTw.png)


When a browser encounters a URL with an “@” symbol, such as https://legit.com@evil.com, it interprets the part before the “@” (in this case, legit.com) as the user credentials (username) for authentication. The actual URL that the browser will navigate to is the part after the “@” (in this case, evil.com).

Therefore, when you enter https://legit.com@evil.com in the browser, it ignores legit.com and navigates to https://evil.com. This behavior can be exploited to create deceptive links that appear to lead to a trusted site but actually redirect to a different, potentially malicious site.

For example:

	•	URL entered: https://legit.com@evil.com
	•	URL loaded: https://evil.com

### Open Redirect in WebView

Now, in summary, we would have an open redirect in the WebView. However, since it is a financial application, I would like to find a way to further impact this vulnerability. Analyzing the WebView that the application was supposed to load, I found in the libs that the page loads a WebView-bridge, which is essentially a bridge for communication/interaction between the page and the application.

### Custom Bridge Implementation

I think you can see where this is going, right?

If we redirect the application to our own server where we have implemented a custom bridge, we can control some calls that the application has implemented. I started rewriting the WebView-bridge library on a web page on my server. In the image below, I show some of the code.

![IMG](https://miro.medium.com/v2/resize:fit:618/format:webp/1*pAOWIGpdpNeQ5Eget6jAKA.png)

![IMG](https://miro.medium.com/v2/resize:fit:1240/format:webp/1*feK2kZoKJio5EcbxtLl_aQ.png)

![IMG](https://miro.medium.com/v2/resize:fit:718/format:webp/1*Z5-iW2sJQt4DjQOqCNkaAg.png)

### JavaScript Bridge

A JavaScript bridge acts as a communication channel for bidirectional data exchange between web content and the native application. iOS and Android WebViews support communication through JS bridges.


![IMG](https://terra-1-g.djicdn.com/71a7d383e71a4fb8887a310eb746b47f/cloudapi/V1.1/JSBridge%20%E4%BB%8B%E7%BB%8D%E5%9B%BEen.png)


### Exploration

Given this information, I created a POC (Proof of Concept) to exploit this vulnerability. Here are the steps:

1. **Set up a server**: I set up a server to host my custom WebView-bridge.
2. **Redirect the application**: By modifying the deeplink path to include `@my-custom-server.com`, I redirected the WebView to load my server instead of the intended URL.
3. **Implement the custom bridge**: On my server, I implemented a custom WebView-bridge that mimicked the original bridge but with added capabilities to intercept and control the communication between the web content and the application.
4. **Test the vulnerability**: With the custom bridge in place, I tested various interactions to see what data I could access and what actions I could perform.

### Results

The custom bridge allowed me to intercept sensitive information and perform actions that should not have been possible, demonstrating a significant security vulnerability in the application.

1. **user data**: We can use the javascript bridge to get the user data, we can easily save this data by sending it to our server and storing it. As a visual example, I gave an Alert() on the data received from the javascript bridge

![datauser](https://miro.medium.com/v2/resize:fit:530/format:webp/1*o0pZjce0sTO-OYlmLLS4AA.png)

2. **PIX transfers**: To make it even more critical, I decided to use the application’s API, which we could interact with to request a transfer in the background after loading the page. It required a second implementation in the bridge we had developed, but in the end, it was possible.

![IMGBRIDGEPIX](https://miro.medium.com/v2/resize:fit:778/format:webp/1*3lX02Y_UweJ-AjzR8dGqtQ.png)

![IMGPIx](https://miro.medium.com/v2/resize:fit:788/format:webp/1*VYMt3s7VyAA3Kq52F5idgQ.png)

### Conclusion

This vulnerability highlights the importance of proper validation and security measures when handling deeplinks and WebViews in financial applications. Developers should ensure that all parameters are properly sanitized and that WebViews are configured to prevent such exploits.

For more information, you can check my profile: (@thalyson.sec)(https://medium.com/@thalyson.sec)
