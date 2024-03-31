---
layout: post
title:  "Improving consistency in distributed applications using Saga pattern with Orkes (Netflix) conductor" 
date:  2024-01-28 11:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["BackEnd","Architecture","Cloud"]
---

* TOC
{:toc}

### Intro
Microservice architecture has become the standard for most complex systems.
While it addresses many issues, it also introduces certain complexities.
One such complexity is maintaining ACID principles in distributed flows across multiple services.  
A basic example involves a product purchase flow, requiring:

- Inventory verification (Inventory Service)
- Payment processing (Payment Service)
- Product shipment submission (Shipment Service)

Consider a situation where both the inventory verification and payment processing succeed, but your request is denied by the Shipment Service.  
Assume, for instance, the rejection stems from a sudden change in shipping policies that disallows delivery to the provided customer address, rather than from some technical reason,
which theoretically can allow us to retry instead of failing a flow.  
In such circumstances, you would likely need to process a refund and possibly inform the Inventory Service that the item is available again.  
This scenario highlights a particular challenge, but that there may be additional, similar situations. 
Therefore, a more general, infrastructure-level solution is necessary to manage failures within distributed workflows effectively.

### Saga Pattern
In simple terms, the Saga pattern is utilized to maintain consistency in distributed applications through coordination and failures management. 
A Saga consists of a series of transactions that are initiated either by a central coordinator (Orchestrated) or through a reactive approach by responding to events (Choreographed).  
When any transaction within the Saga fails, a compensation transaction is activated to revert the application back to a consistent state.  
Taking our example (assuming we're using a coordinator), if the shipment service transaction fails, 
the coordinator will initiate a refund transaction with the payment service and update the inventory status with the inventory service.

### Orkes (Netflix) Conductor as infrastructure level solution
Conductor is a microservice orchestration tool created by Netflix and later forked by Orkes.  
Given the right circumstances, it is a good solution if you're looking to implement a generic, infrastructure-level solution to orchestrate your distributed flows.  
It supports creating and managing complex applicative flows, including timeouts, retries and a lot of useful features, among them - failure handling.  
With that being said, it is not without caveats, starting from maintaining additional infrastructure to the need for careful workflow design (e.g, not passing large payloads between workflows).

### Compensation transactions
Conductor offers a concept of a <a href="https://orkes.io/content/error-handling" target="_blank">failureWorkflow</a>.  
Failure workflows can be added to your distributed workflows and triggered if one of the transaction in the Saga did not successfully complete.  
Conductor interface provides a clear visibility over the workflow itself and the routes it can take whether succeeded or failed, making it a solid choice if you're looking for a
battle tested infrastructure level microservice orchestration solution.
