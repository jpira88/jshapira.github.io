---
layout: post
title:  "HTTP compression: reduce up to 90% of HTTP response size with Gzip"
date:  2016-06-27 18:33:28 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["#BE", "#Performance"]
---

Speed is one of the most important (if not the most important) aspects of a quality application.
Among others, application speed is effected by the speed of the HTTP requests,
which effected by things we can’t always (network connection) and can (responses size, structure etc’) control.
HTTP compression provides a neat method to control response size and reduce the amount of time it takes for an HTTP request to complete.

Gzip is one of most popular compression utilities and can reduce your response size by up to 90% (You can see a compression list here). One of the nicer parts about Gzip is that from server point of view, it’s relatively easy to setup on most of the modern servers, and from client point of view you literally don’t need to do anything. All of the modern browsers support it, 
you can see it if you open the requests details on your network tab and look at *Accept-Encoding*:

![Browser Supports GZIP](/assets/post-images/2022-01-06-reduce-with-gzip/2022-01-06-reduce-with-gzip-1.jpg)

What basically happens is that the HTTP request notifying the server that it can accept Gzipped content, and the server, if configured, Gzipping the content before returning the response.

### Configure Apache
Apache supports 2 types of compression options, mod_deflate and mod_gzip. We’ll use mod_deflate since this mod is actively maintained, comes out of the box and easy to setup.

As explained, mod_deflate comes right out of the box in latest Apache installs (at least in Windows installer and trough Ubuntu apt-get), so you don’t need to install anything. To be on the safe side you can check if the mod is available by running:

`apachectl -D DUMP_MODULES | grep deflate`

You should see something like:

`deflate_module (shared)`

After we’ve validated that the mod is active, add this to your .htaccess file:

{% highlight shell %}
<IfModule mod_deflate.c>
        <IfModule mod_filter.c>
                AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css application/x-javascript application/javascript application/ecmascript application/rss+xml application/xml application/json
        </IfModule>
</IfModule>
{% endhighlight %}

This is pretty much self explanatory, we’ve just added multiple content types to be compressed after checking that deflate and filter mods are enabled. Note that if you need to support really old browsers you can use Apache’s BrowserMatch directive:

`BrowserMatch [HTTP-Agent-Regex] gzip-only-text/html`

You can also set DeflateCompressionLevel directive to control the compression.

`DeflateCompressionLevel [1-9]`

The higher the value the better the compression at cost of more CPU.
Now restart Apache and that’s it.

### Configure Nginx

In order to configure nginx you should edit your nginx.conf file : 

{% highlight shell %}
gzip on;
gzip_types text/html text/plain text/xml text/css application/x-javascript application/javascript application/ecmascript application/rss+xml application/xml application/json
{% endhighlight %}

Also on Nginx you can disable gzip for certain browsers and control the compression level:

{% highlight shell %}
gzip_disable [HTTP-Agent-Regex];
gzip_comp_level [1-9];
{% endhighlight %}

Now restart nginx and everything should be working.

### IIS
I’ve never really configured gzip for IIS, but a quick google yield this [highly voted answer](https://stackoverflow.com/questions/702124/enable-iis7-gzip){:target="_blank"}. 

### How can I tell my response is compressed?

You can make sure your response came back compressed if you open your network tab and look at the response headers:

![Browser Supports GZIP](/assets/post-images/2022-01-06-reduce-with-gzip/2022-01-06-reduce-with-gzip-2.jpg)

You can also see the before/after compression size in the main HTTP Requests view:

![Browser Supports GZIP](/assets/post-images/2022-01-06-reduce-with-gzip/2022-01-06-reduce-with-gzip-3.jpg)

If you look at the size column, you’ll notice a black and greyed out numbers. The grey number represents the size of the actual size while the grey one represents the compressed size. 