---
layout: post
title: "Azure Architecture Patterns"
author: "Hersh Bhasin"
comments: true
categories: AZ-300 Azure-Architecture-Patterns
published: true

---

Video: https://www.youtube.com/watch?v=sUfmavlYx68

MS: Pillars of Software Quality https://docs.microsoft.com/en-us/azure/architecture/guide/pillars

Framework: https://github.com/azure/fta-architecturalreview

![](..\assets\best10.PNG)

![](..\assets\best9.PNG)

![](..\assets\best1.PNG)

![](..\assets\best2.PNG)

![](..\assets\best3.PNG)

Shared Services Examples

![](..\assets\best4.PNG)

![](..\assets\best5.PNG)

![](..\assets\best6.PNG)

![](..\assets\best7.PNG)

![](..\assets\best11.PNG)

> mnemonic: FGVS: F*ing Great Valet Hosting

1. Federated Identity Pattern:  Example: Logging into Azure AD/ ADFS into any kind of service.

2. Gatekeeper Pattern:  A security instance acts as a gatekeeper. Example: a reverse proxy layer (Application Gateway, Front Door), with web application firewall (waf) enabled, that protects my application from attack.

3. Valet Key Pattern. Example: File uploads. Instead of giving you a storage key, I give you a shared access signature to only upload to that one blob. Refer https://hershbhasin.com/2018-08-03/azure-storage

4. Static Content Hosting Pattern:  Example: a CDN handling caching of content

![](..\assets\best12.PNG)

**App Scalability Patterns**

>  mnemonic: Competes with CQRS

1. Queue Based Load Levelling Pattern:  you are in a Q
2. Competing Consumers Pattern:  Implement when  #1 fails: if I have a really long Q and only 1 instance processing it, the Q becomes a bottleneck: then add more instances of consumers
3. CQRS pattern

**Reliability & Performance Patterns**
![](..\assets\best13.PNG)

**Throttle & Break Cache**

1. Throttling:  Design to a certain performance level. Over and beyond this level, fail with "too many requests".  APIM policy can be used to throttle excessive requests.
2. Circuit Breaker Pattern:  Opposite of retry pattern. I know that service is down; no point retrying as it is never going to work. Wait when it is back up and then retry.
3. Cache Aside Pattern:  Redis. Look to see if data is in cache first before hitting storage. If data not in cache, get from storage  and also  put in cache. If another instance then writes to the storage, notify  all cache consumers that cache has changed and flush the cache. Effect: you will never have stale data because the next request that will be made will  read fresh data from storage and will put fresh data back in the cache.



# References

https://www.youtube.com/watch?v=H0-_XCv7-A4

Video: https://www.youtube.com/watch?v=sUfmavlYx68

MS: Pillars of Software Quality https://docs.microsoft.com/en-us/azure/architecture/guide/pillars

Framework: https://github.com/azure/fta-architecturalreview