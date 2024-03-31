---
layout: post
title:  "Session locking – big bad and sometimes (or mostly) unnoticed until it’s too late con of long polling"
date:  2016-03-03 18:33:28 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["BackEnd"]
---

* TOC
{:toc}

### Intro
In this post I’m not going to discuss long polling VS short polling VS sockets, I’m also not going to say anything against (or in favor) of long polling. I assume that anyone reading this already done their research and considerations and aware of most (or all) of the pros and the cons of each method. I just wanted to share something that is not mentioned in most of the discussions i heard about the polling methods: PHP’s sessions locks.

By default, PHP uses files for storing session data. This means that in case of long polling, by default, the relevant session file will be locked, so any additional incoming request from the same user will have to wait until the session file will be unlocked. 

### “So what? This is a price I’m willing to pay”
Follow me on this one – you have a client app that creates a long polling connection, waiting for the response, even displaying a nice loading screen. So far a good (or ok-eish) user experience. But what if the user will say “Ok then, I know this is going to take a while. So I’ll just open a new tab while this is loading and do other stuff”? Well, if you were not prepared for this, then you are, gently put, screwed. The user won’t be able to access your application, and I’m not talking about some nice ‘Please wait, in process’ screen, I’m talking about server-takes-forever-to-respond scenario.

I’m probably a little bit over-dramatic here. But this will happen, and if it isn’t ok with you, than it will be a problem that may be a bit expensive to fix, depending on when you discover it. 


### “On second thought, maybe it’s a price I’m not willing to pay”
You can take different approaches on fixing this, depending on how much you are willing to invest (in time or money), the stage your project at, your infrastructure and other factors.

- **Explicitly closing the session with session_write_close:**
Simple in theory: After finishing writing data to the session file, you can close it and unblock it for other processes, afterwards you’ll be able to read from the session but not write to it. In practice you probably should take more cautious approach with this one, most backend servers today written on top of frameworks, whether its Laravel, Yii, Cake or anything else. Each of those frameworks ships with components that require session writing privileges, components such as Authentication, Permissions etc’. So make sure you really understand what’s going under the hood of your framework before unblocking the session.

- **Using non-blocking session storage such as a database, redis, memcached or other equivalents, with a framework shipped connector** – Some may argue, but this is my personal favorite for couple of reasons:

    1. Managing session locks by yourself is more prone to mistakes than letting a fully tested and proven framework to do it for you.
    2. It prevents a possible security breach where session file stored in a shared folder and may be accessible to unauthorized 3rd parties.
    3. It’s easier to scale out.

