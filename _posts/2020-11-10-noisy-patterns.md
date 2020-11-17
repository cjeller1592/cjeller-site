---
layout: post
title: Noisy Patterns
summary: Journaling about how a basic search of my syslog led to action (mostly searching Google).
---

I want to get more familiar with security information & event management (SIEM), so I thought the best place to start would be with system logs. I added the syslog to Splunk (which I also want to get more familiar with).

From Splunk's search bar I type out `sourcetype=syslog` and see what happens. 5018 events in one week. Damn. How am I going to dig through all these logs? Go for the low hanging fruit. Splunk has a _Patterns_ tab to find common occurrences in your data. A click reveals the most common appearing around 2000 times. Just about half of the syslog is clogged up by this event. What could it be? Let's take a (redacted) look.

```
my-machine kernel: [10364.424213] [UFW BLOCK] IN=eth0 OUT= MAC=[REDACTED] SRC=192.168.50.1 DST=224.0.0.1 LEN=36 TOS=0x00 PREC=0x00 TTL=1 ID=60184 DF PROTO=2
```
So my firewall is blocking _something_ at a prodigious rate. Where is my router sending packets to? Am I under attack? Let's keep calm and unpack the problem. First question — has anyone else asked this question before? A search lands me on a cozy [Ubuntu forum](https://ubuntuforums.org/showthread.php?t=1886913). JKyleOKC outlines the situation to the OP:

>It looks as if your virtual server is sending a packet out to another address, and receiving a multicast reply that is being blocked and logged. This may be related to your use of dyn.dns; all of the dynamic dns services presumably must maintain a dialog with your machine in order to detect any change of its IP address and correct their own tables. I use a different service myself so cannot say for sure how dyn.dns handles things, but a multicast response seems reasonable.
>The only way I know to tell for sure what is going on is to run "wireshark" on your system and examine the actual packets. The wireshark program can interpret packet contents for you and make things quite a bit more clear. It's in the repositories...

Thanks JKyleOKC (wherever you are). I am going to take the gist of his suggestion and run a packet capture using tcpdump. The thing I want to focus on is traffic from host 224.0.0.1. This will help identify what the router is sending packets to.

```
tcpdump -w splunk.pcap host 244.0.0.1
```

Only 2 packets are captured. Let's read them using tshark.

```
tshark -r splunk.pcap -V
```

When I look at the packet capture, I see that the machine is in fact my router. The protocol it's using is IGMP — Internet Group Management Protocol (this was `PROTO=2` in the log). I'll let [Wikipedia](https://en.wikipedia.org/wiki/Internet_Group_Management_Protocol) give a better definition than I could:

>The Internet Group Management Protocol (IGMP) is a communications protocol used by hosts and adjacent routers on IPv4 networks to establish multicast group memberships. IGMP is an integral part of IP multicast and allows the network to direct multicast transmissions only to hosts that have requested them.

Just like JKyleOKC said, my router is probably trying to doing some kind of multicast transmission that my firewall is having none of. And the problem is that it my router _really_ wants to be friends with everyone on the network. So much so that it's sending hundreds of multicast requests.

How do I get around this noise? Creating a firewall wall to deny traffic going out to the subnet address (244.0.0.1) might help. It's what others on that above forum topic suggest at least.

```
ufw deny to 244.0.0.1
```

Time will tell whether my logs will be cleaned. Well actually, Splunk will tell me. Since Splunk dynamically pulls the syslog and reads those events, I can set up an alert that will send me an email when the event in question exceeds a certain threshold.

No news will be good news.
