---
layout: post
title: Drag and Drop
summary: Learning about Canarytokens by using Glitch & Netlify (and vice versa).
---
One thing that I love about PaaS services is how quickly you can create a proof of concept. Recently I've found out about [CanaryTokens](http://canarytokens.org), a free & easy honeypot tool. In particular, there's a type of token that triggers an alert when your website is cloned. How could I test this out without much hassle? PaaS.

The test website will be made with [Glitch](https://glitch.com). All I have to do is spin up a static site from one of their templates.

![](/images/Glitch site.png)

In a matter of seconds I have a site I can put the token in. Next I created a Canarytoken using the auto-generated URL Glitch gave my site.

![](/images/Canary.png)

Once the token is created, I receive some JavaScript that you add to the site. It's just a matter of adding the code to my site's JavaScript file the site using Glitch's in-browser editor.

![](/images/Glitch editor.png)

Now that my site is all set up, I'll play the role of the attacker. I want to clone the site to trick people to click stuff they shouldn't. Anyways, I'll go into my terminal and pull the site down recursively using wget.

```
wget -r https://geode-living-jackrabbit.glitch.me/
```

With the site in a local directory, I want to deploy it as a fake website to activate the token. There's an easy to do this. Netlify has a wonderful service called [Netlify Drop](https://app.netlify.com/drop). All you have to do is drag in the folder with your site's HTML, CSS, & JavaScript and Netlify deploys the site for you. (I'm surprised I only heard about it now — it's been around since 2018)

So now I just drag the folder of the Canarytoken site into Netlify Drop.

![](/images/Netlify Drop.png)

As soon as the site is live I receive an email alert that the Canarytoken was triggered!

![](/images/Canary Alert.png)

What's interesting here is that the source IP address belongs to an AWS EC2 instance. This gives us a glimpse into both Netlify and the attacker. Netlify probably uses an automated process to put my site into an available EC2 instance running some version of Linux. All of which happens in the background as I drag and drop a folder into Netlify Drop.

If the Cloud wasn't an abstraction already, PaaS is an abstraction of an abstraction. They not only hide the computer from you but the cloud as well. This can be a good thing. Platforms like Glitch and Netlify manage cloud infrastructure so you can focus on building your app rather than configuring the EC2 instance that hosts your app. While this is already common knowledge, I find it fascinating that this knowledge arose from using Canarytokens.

I also can't help but think of how PaaS makes it easy for phishing sites to be deployed. PaaS also gives an attacker yet another means to obfuscate their whereabouts. We only know that the attacker deployed her site using an EC2 instance from an AWS data center in North Virginia. That doesn't mean she's there. It also doesn't mean she used AWS directly — enter PaaS middlemen like Glitch and Netlify.

And yet these platforms advise against using their services for such purposes — it's laid out in the terms of service. Does that stop people though? I'm curious how Glitch and Netlify try to curb these phishing sites. For one, sites made on each platform without an account expire after a period of time. That's just one control that's simple but effective against driveby phishing attempts. What about phishing from those with accounts? What then?

These questions come from the democratizing power of PaaS platforms like Glitch and Netlify. What does security look like when anyone can host a web app by dragging and dropping a folder?
