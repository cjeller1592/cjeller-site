---
layout: post
title: Experiments by Proxy
summary: Trying to see what blocking Twitter looks like with Squid, and what it can teach us about proxies.
---

I've recently stumbled across squid. No not the animal, thank goodness. The [other kind](http://www.squid-cache.org/) of squid.

Squid is a server I can run in the background on a machine. Like a web server? Sort of. Squid acts as a proxy to my web browser. This means that any time I make a request to a website, it has to go through the Squid server first. In that moment, Squid can use rules I make to decide whether the request goes through. If the website is on my blocklist, for example, the request is denied.

So what does that look like? Say I have Twitter on my block list and I try to go to twitter.com on Chrome. This is what I'd encounter:

![](https://i.snap.as/2YTtF4EO.png)

If I didn't know Squid was running, I'd think Twitter was down or something. What does `ERR_TUNNEL_CONNECTION_FAILED` mean anyway? There's a blog post for that:

> Just to let you know, the ERR TUNNEL CONNECTION FAILED error in Chrome occurs when Chrome cannot create a tunnel connected with the targeted website host. Simply put, Chrome cannot connect to the internet. One of the main causes behind this error is the use of a proxy to connect to the internet. At times, browsing data and Cookies saved in Chrome may also cause the ERR_TUNNEL_CONNECTION_FAILED error to show up. Whatever may the reason be, the methods to fix this error are fairly simple.

Thanks [blog post](https://thegeekpage.com/solved-fix-err_tunnel_connection_failed-error-in-chrome/). So either my browsing data and Cookies are conspiring against Twitter alone, or Squid is working as promised. While Chrome is sparse, trying Twitter on FireFox gives me more information:

![](https://i.snap.as/4dzXQsvw.png)

That's exactly what I'm looking for. The proxy server, Squid, is refusing connections to Twitter (because that's how I set it up).

How strange that these browsers take different approaches when dealing with proxy servers. When I set up Squid with Firefox, it was integrated in the browser settings. Chrome, on the other hand, kicked me off to change my computer's proxy settings. Not sure whether to think Firefox views proxies as a first class citizen on its browser while Chrome brushes it off to the OS. What does that say about each browser's priorities? A question for another time.

Anyways, let's peel back a layer and see what Twitter looks like on Squid's logs. This might give us more information on what happens when Squid blocks a website. Below is from `/var/log/squid/access.log`. It shows my laptop (192.168.50.39) trying to access Twitter:

```
1604799814.830      0 192.168.50.39 TCP_DENIED/403 3982 CONNECT twitter.com:443 - HIER_NONE/- text/html
```

The interesting thing here is the HTTP error message — 403. That usually means that the client (my laptop) is not granted to access to a website (twitter.com) for some reason. Here we are given a rather vague reason — `TCP_DENIED`. Given that we know Squid is running, it makes sense. Squid is denying the TCP connection needed for our browser to connect to Twitter.

But how do I *know* that Squid is doing this? Well, we have to peel back yet another layer. This time I'll sniff things on the packet level to see what happens when I attempt to access Twitter. A little tcpdump will do the trick.

`tcpdump -i any -w twitter.pcap`

This is the packet capture in Wireshark. My laptop is 192.168.50.39 and the Squid server is 192.168.50.102:

![](https://i.snap.as/L80Ip4WE.png)

My inkling about Squid denying a TCP connection is bogus. When you look at the packet capture, a TCP connection occurs within the first three packets! It just happens between my laptop and the Squid server rather than with a web server directly. This is because, shocker, Squid is a proxy server. My laptop doesn't connect to websites directly. Squid does so on my behalf.

Once the three-way handshake occurs, we get this interesting HTTP request method I've never seen before — `CONNECT`. [Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT) has a handy definition for it:

> The client asks an HTTP [Proxy server](https://developer.mozilla.org/en-US/docs/Glossary/Proxy_server) to tunnel the [TCP](https://developer.mozilla.org/en-US/docs/Glossary/Transmission_Control_Protocol_(TCP)) connection to the desired destination. The server then proceeds to make the connection on behalf of the client. Once the connection has been established by the server, the [Proxy server](https://developer.mozilla.org/en-US/docs/Glossary/Proxy_server) continues to proxy the [TCP](https://developer.mozilla.org/en-US/docs/Glossary/Transmission_Control_Protocol_(TCP)) stream to and from the client.

In this case, my laptop (client) asks Squid (proxy server) to tunnel the TCP connection to Twitter (desired destination). I wonder if this is similar tunneling that occurs with VPN's? Instead of tunnelinwe see the 403 HTTP response we analyzed from the Squid logs.

So what does `TCP_Denied` mean given this extra context? What we know is that the TCP connection is already established between my laptop and Squid. So what's being denied? Well, my laptop is asking Squid to tunnel the established TCP connection to Twitter. What does Squid do? Deny that request. Otherwise, if Twitter wasn't on my block list, I would've gotten a 200 HTTP response and been able to access the site. It would look like business as usual.

And that's the potentially malicious side of proxies. As we saw, a proxy server acts as the intermediary between you and the web. There's a lot that can happen between you making a request and the proxy server tunneling the TCP connection to your requested site. I could only imagine tools that mimic [Freedom](https://freedom.to) and similiar services, assuring users that they're only site blockers, but have seedier machinations underneath. They're out there.

Having a proxy can be completely mundane, like with my Squid experiment. On the other hand, proxies can act as an attack vector. They're called man-in-the-middle attacks for a reason. This experiment barely scratches the surface of understanding what proxies do behind the scene. I'd be curious to do similar analysis to see how HTTP & TCP traffic looks different with a nefarious proxy than with a proxy like Squid.
