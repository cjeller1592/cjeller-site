---
layout: post
title: Learning Tools with Other Tools
description: I'm slowing finding that you can best understand particular tools through the lens of others.
---

I'm slowing finding that you can best understand particular tools through the lens of others.

[Nmap](https://nmap.org/), for example, is a powerful network scanning program that I've been getting familiar with lately. The power of Nmap lies in the information it gives you about a network or IP address. So I threw caution to the wind as I used Nmap, scanning my home network and showing my wife that I found her laptop and which ports were open (there were none).

But I learned that with this power comes enormous responsibility. Nmap is mostly used for network reconnaissance in the context of (ethical) hacking. Because of that, discretion plays a paramount role. You want to make sure that Nmap scans fly under the radar of the network you're probing.

This never occurred to me when I started using Nmap. I only cared about scanning all the things, not whether I'd get caught when doing it. This added layer of context brought about a new question.

> What does a *stealthy* Nmap scan look like compared to a _loud_ one?

I knew I could Google these answers without a problem, but I wanted the praxis component.

Enter another tool that could help me solve questions about Nmap.

There's this fascinating piece of software from [Active Countermeasures](https://www.activecountermeasures.com/) called [Honeyports](https://github.com/adhdproject/honeyports). What it does is listen to a port. When an IP address attempts to make a full established TCP connection to that port, Honeyports dynamically blocks the IP address. It's quick to set up — here we have it it running on port 88 on my laptop.

```
sudo python3 ./honeyports.py -h 192.168.50.115 -p88

Honeyports Version: 0.5
I will listen on TCP port number:  88
Honeyports detected you are running on:  Linux
Enter Commands (q=quit f=flush rules l=list rules):
```

Now any IP address that tries to make a full established TCP connection to my laptop at port 88 will be blocked. What does a full established TCP connection mean here? We send out a TCP packet with a SYN flag. Their machine sends back a packet with the SYN-ACK flag. We send an packet with ACK return. That creates the connection to a port that allows data to flow between their machine & our machine. Honeyports, however, prevents that connection from happening — at least in theory.

As I tinkered with Honeyports, I came to find that it served as a way to answer the questions I had about Nmap. If I could use Nmap on a virtual machine to scan my Honeyports laptop and not get blocked, then I found a stealthy scan. If I got blocked, then I found a loud scan.

Let's explore what a stealthy scan means in this context. If Honeyports is supposed to block any IP address that attempts a *full* established TCP connection, then it shouldn't block any IP addresses that attempt a *partial* one. What does a partial TCP connection look like? If we look back at the three-way handshake, the SYN-ACK reply let's us know that a port on a machine is open. But once we get this info, we can choose to hang up. This means we send back a packet with a RST flag instead of an ACK flag. This practice is what's called a SYN scan.

Nmap allows us to do a SYN scan with no problem. To do so, I'll specify the IP address of the Honeyports laptop, port 88, and the SYN scan with `-sS`.

```
nmap -sS 192.168.50.115 -p88
```

Here's the results from the scan.

```
Nmap scan report for adhd (192.168.50.115)
Host is up (0.0014s latency).

PORT   STATE SERVICE
88/tcp open  kerberos-sec
```

Okay, the port is up. Did the VM get blocked by Honeyports? When I go back to my Honeyports laptop, the VM's IP address is not on the blocked list. Awesome! This proves that a SYN scan is indeed a stealthy scan.

So what would a loud scan look like? One Nmap scan I like to use is the version scan. This not only checks to see what ports are open but does its best to grab information about the services running on those ports. To set up this scan in Nmap, I'll do the same thing as before but specify `-sV` instead.

```
nmap 192.168.50.115 -sV -p88
```

As soon as I ran this scan, Honeyports chimes back with this:

```
Blocking the address:  192.168.50.109
Creating a Linux Firewall Rule

I just blocked:  192.168.50.109
('192.168.50.109', 51102) disconnected!
```

The VM got blocked! An Nmap version scan makes a full established TCP connection. Because of this, Honeyports struck with a vengeance. The risk of trying to probe for version information is a loud scan that could get you detected & blocked.

What's great about this experiment is that you can add even more tools into the mix. What if we used tcpdump to examine what happened when HoneyPort blocks an IP address that is attempting an Nmap scan? What more could we learn about tcpdump? About packet capturing? Learning tools with other tools turns learning into an ever iterative and remixable exercise — endlessly exchange one tool for another.

I think learning tools with other tools is something that appealed to me even when I didn't know how to write a line of code. When music was my main pursuit, I found that I learned most about my primary instrument, the guitar, not from playing with other guitarists but other instrumentalists — drummers, bassists, trombonists, violinists. Through playing together I had a better sense of what my instrument's role was.

Perhaps that's why I love doing that with software.
