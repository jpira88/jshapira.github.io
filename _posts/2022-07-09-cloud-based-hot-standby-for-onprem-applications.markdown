---
layout: post
title:  "Cloud based standbys for on Premises workloads without changing your public IP" 
date:  2022-07-09 18:31:29 +0200
categories: jekyll update
author: Jacob Shapira
sticky: "true"
tags: ["Architecture", "Cloud", "BackEnd"]
---

* TOC
{:toc}

### Intro
Some workloads rely heavily on having a specific public IP addresses.
For example:

* Workloads with emailing features, where the IP reputation is extremely important to get high delivery rates.
* Workloads in which access to other systems (partners, 3rd party etc') based on IP whitelisting.
* Workloads providing access for field consumers which rely on direct IP address and not on a DNS Server, delivery trucks sending GPS data for example.

Let's assume your team/group is developing an on premise workload which implements both of the examples above.
Now you've been tasked with designing a standby, or even migrating the entire workload to the cloud.

One of the requirements is to maintain your existing IP address to keep all of your existing integrations intact.

Luckily, in recent years, all the main cloud providers (AWS, Azure, Google Cloud) introduced BYOIP (Bring your own IP) feature.

With this feature, you can basically reuse and remain the owner of your IP range, but allow the cloud provider to advertise it.
It means you can prepare a standby for your workload, may it be cold, warm or hot, and if the need arise, instantiate the standby
workload and attach your existing IP range to it, making the process completely transparent to all existing parties consuming/producing data
from/to your workload.


### Prerequisites
The process of bringing your own IP and the prerequisites are pretty similar
among all the providers.

#### IP range registered with a RIR
The main requirement is that the address you are importing must be registered with one of the following RIRs (Regional Internet Registry):

- AFRINIC (Africa)
- APNIC (Portions of Asia and Oceania)
- ARIN (North America and some Caribbean Islands)
- LACNIC (Latin America)
- RIPE NCC (Europe, Central Asia, Middle East)

#### Minimum IPv4 address range
The address range must be no smaller than a /24 so it will be accepted by Internet Service Providers.

#### Other provider specific requirements
On top of the generic requirements, each provider enforces additional specific requirements, such as limit on the amount of ranges you can bring,
validating reputation history of the address and more.

For your convenience here are direct URLs to each provider step-by-step guide to bring your own IP.

- <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-byoip.html" target="_blank">AWS BYOIP</a>
- <a href="https://cloud.google.com/vpc/docs/using-bring-your-own-ip" target="_blank">Google Cloud BYOIP</a>
- <a href="https://docs.microsoft.com/en-us/azure/virtual-network/ip-services/create-custom-ip-address-prefix-portal" target="_blank">Azure BYOIP</a>


