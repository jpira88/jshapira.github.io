---
layout: post
title:  "Improve MTTR's with observability platforms" 
date:  2022-10-23 18:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["#Architecture", "#Cloud", "#BE", "#Devops"]
---

Ability to quickly resolve production issues is one of the key factors in customer satisfaction.  
Not the mention it saves us resources and debugging time.  
Cloud native and distributed systems are especially hard to debug due to high amount of moving parts.  
An error in a flow can originate in a one of N containers of a specific service, in a flow composed of N different services, 
and the erroneous container might as well be dead already due to different reasons (HPA, a crash, etc').  
Debugging is hard, reproducing is sometimes harder.

### Observability platforms to reduce the pain
Tools such as Rookout, Lightrun or Helios can greatly reduce debugging complexity and MTTR's (Mean time to response/repair/resolve/recover).

For example,
You can set ad-hoc virtual non-breaking breakpoints in a live production environment.
You'll be able to get information (such as variables, context, evaluate expressions and basically almost everything you'll get from a debugger) from any service, down to specific line of code, if it was triggered during a flow.
Thus making debugging much easier.

You can also get live visual representation of how components in your flow interconnects with each other, down to API payloads, kafka messages and more.
You can even reproduce entire flows with a click of a button.

Basically, you take the power and flexibility of local debugging and issue investigation and apply it on a production (or test/staging) environment.

### Pricing
The pricing of most of these tools a relatively affordable for most companies.
Obviously the exact price depending on the tool itself and the package you select, but in general it starts at around 700$/mo at the time of writing this post.

### Demos & Choosing the right tool for you
You can book an official demo in the websites, but if you'd like a quick overview here are some useful youtube videos:
- <a href="https://www.youtube.com/watch?v=5hR17z4Qm3g" target="_blank">Lightrun</a>
- <a href="https://www.youtube.com/watch?v=Tnsd_65jQLY" target="_blank">Rookout</a>
- <a href="https://www.youtube.com/watch?v=OI5oYxTCiV0&t=957s" target="_blank">Helios</a>




