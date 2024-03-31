---
layout: post
title:  "F-Strings formatting is not only the most modern approach, it's also the most performant"
date:  2022-06-05 18:33:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["Performance", "BackEnd", "DataEngineering"]
---

* TOC
{:toc}

### Intro
There are many ways to format strings in python, concatenation, modulus, ordered, named, and f-strings.   
Making sure that all python3+ applications in your ecosystem use f-strings is not only
a matter of standardizing the code base and enforcing best practices among all teammates.  
It's also a matter of performance, different string formatting methods can swing performance in up to 55%. 

This is a high number, especially in applications with heavy dependency on string formatting.  
Some may argue that saving a 1-2 of seconds per millions of actions is not that important,
but :
* Faster is always better than slower.
* Think about scaling, eventually it's a numbers game. String formatting is a CPU bound task,
higher CPU utilization means spending more money, or hogging resources from other services and applications.

### A little test
{% highlight python %}
import time

my_str = "converted_string"
my_int = 100
repeat = 10000000

concat = lambda: "string: " + my_str + ", int: " + str(my_int)
modulus = lambda: "string: %s, int: %d" % (my_str, my_int)
ordered = lambda: "string {}, int: {}".format(my_str, my_int)
named = lambda: "string: {s}, int: {i}".format(s=my_str, i=my_int)
f = lambda: f"string: {my_str}, int: {my_int}"

def test(fn):
    start_time = time.time()
    [fn() for _ in range(repeat)]
    print(time.time() - start_time)


test(concat)
test(modulus)
test(ordered)
test(named)
test(f)
{% endhighlight %}

### And the results
{% highlight shell %}
3.7484169006347656
3.6047866344451904
3.6852612495422363
4.343647003173828
2.4641149044036865
{% endhighlight %}

As you can see, f-strings formatting is faster by 37.5% - 55% than other methods.
This is insignificant number in small deployments, but will definitely reduce execution time and budget when scaled.
