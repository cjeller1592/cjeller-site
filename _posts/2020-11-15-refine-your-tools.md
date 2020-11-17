---
layout: post
title: Refine your Tools
summary: Continuing the experiment from the Noisy Patterns post.
---
_This is an update from the [Noisy Patterns post](/2020/11/10/noisy-patterns)_

I haven't received an email notification from Splunk since I made the rule in my firewall.

No news is good news right? Well, no news could mean bad news too. Maybe the email didn't go off. But it could be a problem on my end. Since the event took up a big chunk of the machine's syslog (2000 times in one week), I configured the alert so that it'd send an email when the event appeared 100 times. That way I could catch it at the beginning. What if the event only occurred 50 times? 99 times? I had to investigate the logs.

Back to Splunk I went. Here's the event I was looking for in the syslog:

```my-machine kernel: [10364.424213] [UFW BLOCK] IN=eth0 OUT= MAC=[REDACTED] SRC=192.168.50.1 DST=224.0.0.1 LEN=36 TOS=0x00 PREC=0x00 TTL=1 ID=60184 DF PROTO=2
```

How could I break this down into something that Splunk can search for? I created a query for an event in the syslog where the source address is the router's (192.168.50.1) and the destination address is the all-hosts group's (224.0.0.1).

```sourcetype="syslog" SRC="192.168.50.1" DST="224.0.0.1"
```

Nothing showed up in the syslog from this search. Great.

But couldn't I save doing the search if I set the email notification to send with a single instance of the event in the syslog? Definitely. I think I had the overwhelming logs at the forefront of my mind when I set the notification to go off at 100 occurrences. Even one occurrence of the event would've been too many. Guess I'll update that.

This goes to show the never ending process of refining your tools. To paraphrase [John Strand](), the thing you have to be scared of isn't false positives but false negatives, so refine your tools!
