---
title: "Race Condition: Some Real Cases"
date: 2024-05-25
# weight: 1
# aliases: ["/first"]
tags: ["web", "race-condition", "bug-bounty", "en"]
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


## Understanding Race Conditions

Race conditions are a prevalent vulnerability closely linked to business logic flaws in web applications. They arise when websites process concurrent requests without proper safeguards, leading to multiple threads interacting with the same data simultaneously. This interaction can cause "collisions," resulting in unintended application behavior. Malicious actors exploit these race conditions by carefully timing their requests to create intentional collisions for nefarious purposes.

For those interested in diving deeper into race conditions or practicing in controlled environments, numerous online labs and resources are available.

-> [Portswigger Race Condition](https://portswigger.net/web-security/race-conditions?source=post_page-----598b260ab3f0--------------------------------)


## Introduction to Real-World Scenarios

In this post, I'll share some race condition scenarios I encountered during bug bounty hunting in 2023 and 2024. These findings were inspired by the insightful research of [@albinowax](https://twitter.com/albinowax).

## Scenario 1: Exploiting Refund Mechanisms

### The Vulnerability

This scenario represents the most critical vulnerability I discovered, allowing for the exploitation of the application's refund feature.

### The Process

1. **Service Subscription A**: Users could choose between two plans:
   - Plan A: R$ 49.00
   - Plan B: R$ 450.00

   ![](https://miro.medium.com/v2/resize:fit:792/format:webp/1*n0YtGsG1EQmXBCHTlsksZw.png)

2. **Refund Policy**: After subscribing, customers could request a refund for the full amount paid.

3. **Exploitation Method**: 
   - I used the turbo intruder tool, selecting the race condition option.
   - Multiple cancellation requests were sent simultaneously.
   - Several requests returned a 200 status, indicating successful processing.


- **Plan A**
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*RW1BVcjB85vPGup8HxGZHw.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*yBGcjXVdquA3hX7kAwb2lQ.png)

- **Plan B**
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Y7ES0NRfvS2BE8M5Gbxtsw.png)
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*rbkzJFoT9VR3yneBgAIA1g.png)



### The Result

- Initial deposit: R$ 50.00
- Service charge: R$ 49.00
- Outcome: Multiple refund deposits were processed
- Final balance: R$ 981.00


1. **Plan A**

![planA1](https://miro.medium.com/v2/resize:fit:550/format:webp/1*Bb13lPMc2gFmbQILlZKSIQ.png)

![planA2](https://miro.medium.com/v2/resize:fit:584/format:webp/1*4R5Z25PQEBTWzOnOeTlV0A.png)

1. **Plan B**

![planB](https://miro.medium.com/v2/resize:fit:608/format:webp/1*jc9QbolLh8oCoDLd4w9Ciw.png)


## Scenario 2: Bypassing Deposit Limits

### The Context

This case, while not rewarded due to its occurrence in a sandbox environment, provides valuable insights into potential exploits.

### The Vulnerability

- The application had a daily deposit limit of $1,000.
- Using the turbo intruder, I successfully bypassed this limit.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*FV8SdB8yhwU_tvXGqa2yyw.png)
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*iGsbwuSQ5v91UNSES9tHHg.png)

### Demonstration

I replicated this bypass on multiple accounts, consistently circumventing the daily limit. Even when the application displayed a message indicating the limit had been reached, the exploit allowed for additional deposits.

![](https://miro.medium.com/v2/resize:fit:804/format:webp/1*MSjsGV_KzLDgs1-sb-svHA.png)

## Conclusion

These scenarios highlight the critical importance of proper concurrency handling in web applications. Race conditions, when left unaddressed, can lead to significant security breaches and financial exploits. Developers and security teams must prioritize robust safeguards against these vulnerabilities to ensure the integrity of their systems.

## Race Condition Bountys

![](https://miro.medium.com/v2/resize:fit:1134/format:webp/1*05rAAVNpoPrNQkp5GGk68A.png)