---
layout: post
title: The Map of a Map of The Place
summary: Getting help from the Borgesian Infrastructure as Code service Terraform to better understand Microsoft's Azure.
---
Lately I've been wanting to learn more about cloud computing. So, for a reason unknown to me, I picked Microsoft's Azure to start with. 

Along those beginning steps I also picked up learning an Infrastructure as Code tool called [Terraform](https://www.terraform.io/). With Terraform, to put it simply, you define the cloud infrastructure you want in a file and Terraform brings that blueprint to reality. 

It's been fascinating to learn both in parallel. One thing in particular that's stuck with me is that Infrastructure as Code brings out, well, the underlying infrastructure of the cloud. 

Terraform is opinionated in how you arrange your blueprint — first you declare a resource group, under that resource group you declare the virtual network, under that is the subnet, then you create your public IP address for your virtual machine, and so on. That hierarchical mapping helps me visualize how cloud services relate to and work with each other. I couldn't get that from the Azure dashboard alone. Using Terraform acutally helps me better digest the dashboard. Funny how [one tool can help you learn another](https://cjeller.site/2020/10/28/learning-tools-with-other-tools).

So this week I managed to create a simple cybersecurity lab in Azure using Terraform — a virtual network with a Kali Linux box and an Ubuntu box. It's been fun to spin the lab up and try some Kali tools on the Ubuntu box, whether that's mapping out the private network using Nmap or cracking the Ubuntu box's SSH password using Hydra. All you need is Terraform, an Azure account, and the Azure CLI. [Feel free to clone the repo and give it a shot](https://github.com/cjeller1592/terraform-azure-lab)

I'm looking forward to learning more about the cloud and Terraform. Part of me cannot shake the Borgesian vibes I get from Infrastructure as Code. The Terraform file is the map of your cloud infrastructure and yet it also is your cloud infrastructure — if you change the file and run Terraform again, those changes will show in your cloud infrastructure. It is both the map of a place and the place at the same time. Borges put it best in 74 years ago in one of his shorter short stories, _The Exactitude of Science_ (trans. H. Hurley):

> .... In that Empire, the Art of Cartography attained such Perfection that the map of a single Province occupied the entirety of a City, and the map of the Empire, the entirety of a Province. In time, those Unconscionable Maps no longer satisfied, and the Cartographers Guilds struck a Map of the Empire whose size was that of the Empire, and which coincided point for point with it. The following Generations, who were not so fond of the Study of Cartography as their Forebears had been, saw that that vast Map was Useless, and not without some Pitilessness was it, that they delivered it up to the Inclemencies of Sun and Winters. In the Deserts of the West, still today, there are Tattered Ruins of that Map, inhabited by Animals and Beggars; in all the Land there is no other Relic of the Disciplines of Geography