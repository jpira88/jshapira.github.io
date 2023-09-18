---
layout: post
title:  "Anti-Corruption layer is not your way around legacy or badly designed implementation"
date:  2022-07-07 18:30:29 +0200
categories: jekyll update
author: j.shapira
tags: ["Architecture"]
---

So you and your team have been tasked with adding a set of new capabilities to an existing domain, a service or a sub system.  
You research the existing implementation, APIs, databases and technological stack and come to a conclusion that it will be extremely
difficult to extend existing code and capabilities to support those new features.

One of the more experienced team members suggests a valid approach -   
"The stack is outdated," he says, "a badly designed monolith, and the only guy that could understand what's going on quit 2 years ago,
lets create anti-corruption layer, so we won't corrupt our new code with this legacy mess."

The idea is good, the approach is valid, but it has nothing to do with anti corruption layer.  
The suggested approach can be implemented with adapters or facades patterns, which are among the most important patterns in existence.
Anti corruption layers can be composed out of many facades and adapters, but it's not ACL's job to cover up for interfaces you
can't or prefer not to deal with.

#### ACLs should prevent concepts intrusions and mismatches between different bounded contexts
A bounded context is a term in domain driven design which represent a sub system.  
Based on this understanding, lets examine a simplified example of a cyber investigation platform.

[![Simplified Cyber Platform]({{ site.baseurl }}/assets/post-images/2022-08-07-acl/simplified-cyber-platform.JPG)]({{ site.baseurl }}/assets/post-images/2022-08-07-acl/simplified-cyber-platform.JPG){:target="_blank"}

As you can see in the example, each context operates on need to know basis.
* Analytics context needs to know about cyber events repository but should not be aware of the collection process,
which in return decouples the analytic context from understanding and caring what happens in collection context.  
This applies not only to technological perspective (such as backward compatibility, for example), but also from managerial
point of view. The only thing that should be shared between teams responsible for data collection and analytics is
the concepts of events. 


* Same example applies on investigation context, teams working in on this sub system should be only familiar with
investigation artifacts and not how they are collected, or which models applied to produce them.

#### Try to say it out loud
If you feel like this concept is too high level, over-engineered or not necessarily practical.
Try to say it out loud, pretend you're one of the engineers responsible for the investigation context and conducting
a meeting with your teammates. Referring to "investigation artifacts" will feel perfectly natural, it's a known and natural
concept in investigation context. Now try to discuss DS models or which message queues will be used to gather the events, and you'll
see how it seems extremely out of context.

