---
layout: post
title:  "Leverage AWS Direct Connect to use AWS services without exposing your data to the public internet" 
date:  2023-03-26 11:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["DataEngineering", "#BE", "#Performance", "#Cloud", "#Architecture", "#Devops"]
---

While there are a lot of tools to secure your data while being transferred over the internet,
the most secured method of them all is not to do it at all.  
Obviously this approach limits us in many ways, including (but not limited to) leveraging the power of public clouds.

Luckily, AWS (and GCP, but it's out of scope for this post) provides us with a way to use its services without transferring the data over the public internet via AWS Direct Connect.

The general idea of Direct Connect is to create physical connections between AWS Backbone network and your data center via AWS Direct connect location partner.


[![AWS Direct Connect]({{ site.baseurl }}/assets/post-images/2023-03-26-direct-connect/direct-connect-general-arch.JPG)]({{ site.baseurl }}/assets/post-images/2023-03-26-direct-connect/direct-connect-general-arch.JPG){:target="_blank"}

As shown in the diagram, you can physically connect your data center to a router located in the direct connect partner. 
From there, the partner will cross connect your router to AWS router connected to AWS backbone. 
After the physical connections are done, you need to create logical connections via VIFs (Virtual interface) in order to access
AWS public services (such as S3) and your private VPCs or a transit gateway.


#### Selection a connection
You can select 2 types of connections according to your needs and limitations:

| Connection Type | Physical network equipment | Available bandwidths | Hourly Pricing |
| ------- |---------------------------| ------- | ------- |
|  Hosted connection | Usually belongs to the service provider and managed by it | Ranging from 50Mbps to 10Gbps | Higher than a dedicated connection |
|  Dedicated connection | Usually your own          | 1, 10 and 100Gbps | Lower than hosted connection |


#### Creating VIFs and directing traffic to the correct
You can have 3 types of VIFs: public, private and transit VIFs.  
Public VIF for AWS public services such as S3, private VIF for your VPC and transit VIF for a transit gateway.  
Next, in order to correctly direct traffic, you should associate each VIF with a VLAN tag, so the traffic can be routed to the
correct routers on AWS.  

A VLAN tag is a 4 byte tag added to the ethernet frame in networks configured to use VLANs.

You should use the same VLAN tags on your end, as illustrated below:

[![AWS Direct Connect]({{ site.baseurl }}/assets/post-images/2023-03-26-direct-connect/direct-connect-with-vlans.JPG)]({{ site.baseurl }}/assets/post-images/2023-03-26-direct-connect/direct-connect-with-vlans.JPG){:target="_blank"}


This is a high level diagram of how things work. 
You can find a neat <a href="https://docs.aws.amazon.com/directconnect/latest/UserGuide/getting_started.html#ConnectionRequest" target="_blank">step-by-step guide in AWS documentation</a>.


#### Not only for security
Security is not the only benefit of using direct connect. 
You can also get increased bandwidth throughput and a more consistent network performance than what can be achieved with a public internet connection.
 
