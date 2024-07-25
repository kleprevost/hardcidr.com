---
author: ["Kyle LePrevost"]
title: "Salad Vulnerability Disclosure (July 2024)"
date: "2024-07-25"
description: "Salad customers were given full access to a Chef's internal network on Salad versions 1.6.0 and earlier. It was possible for a customer to deploy a malicious container that could quickly scan and exploit vulnerable devices on a Chef's home network."
summary: "Salad customers were given full access to a Chef's internal network on Salad versions 1.6.0 and earlier. It was possible for a customer to deploy a malicious container that could quickly scan and exploit vulnerable devices on a Chef's home network."
tags: ["cybersecurity", "saladcloud", "salad", "salad.io", "vulnerability", "vast.ai", "vulnerability disclosure"]
categories: ["cybersecurity"]
ShowToc: true
TocOpen: false
---

_Salad Cloud was notified about this vulnerability on June 26th and the issue was fully remediated on July 19th with the network upgrade to Salad agent version 1.6.1. Thanks to the Salad team for prioritizing the security of their "Chefs"!_ 

# Context

I've been playing around with a startup cloud service called "Salad Cloud" recently - my last [blog post](https://hardcidr.com/posts/saladcat/) was about how I built a crazy fast hashcat cluster using their service. Salad allows "gamers" to rent out their GPUs on their service in exchange for gift cards, cash, and digital currency. Salad uses that compute power to run a distributed PaaS cloud - as a customer, you just provide Salad a Docker container image and hit "deploy". It works surprisingly well and I've never seen anything quite like it in the industry before. 

Salad has invested a lot of time and energy into the security and privacy of customer container workloads that are deployed onto their network. They have a pretty clever architecture that makes it very difficult to tamper or snoop on what customers are running. Overall I couldn't find any major flaws in how Salad protects customer workloads - honestly it's pretty impressive and might deserve a deeper dive blog post. However, Salad also needs to make sure they protect the backend nodes on it's distributed compute cloud. These nodes aren't Salad-owned servers in a datacenter somewhere - they are PCs that are being run out of residential households - so the threat modeling needs to be approached from a different perspective. 

Just from a terminology perspective - a "customer" is someone paying Salad to deploy containerized workloads, and a "Chef" is someone who is renting their gaming PC out to the Salad network. 

# The Vulnerability

Salad customers were given full access to a Chef's internal network on Salad versions 1.6.0 and earlier. It was possible for a customer to deploy a malicious container that could quickly scan and exploit vulnerable devices on a Chef's home network. This process could have been fully automated and was proven to only cost a couple dollars per day to pull off. 

As a proof of concept I was able to deploy a Kali Linux container image onto the Salad network. This container image contained a reverse shell that allowed for interactive access into the WSL/containerd environment on the "Chef" node. I was able to run NMAP, Metasploit, and other tools from within the Kali environment against the network of the "Chef" without triggering any type of detections from Salad. I was able to perform this type of reconnasaince against several Chefs undetected. I saw quite a few concerning things during my time scanning: 1) open admin interfaces for IoT devices, 2) lots of ancient devices (found a Windows XP desktop!), and 3) mostly default router configurations. 

The fact that people at running vulnerable devices on their home networks wasn't surprising. The fact that I was able to: 1) deploy an obviously malicious container workload, 2) perform obviously malicious things in the container environment, and 3) go completely undetected WAS surprising. 

# The Report and Fix

I notified Salad of this vulnerability on June 26th. I heard back from Salad on July 3rd, then they rolled out version 1.6.1 which fixed the issue on July 19th. 

I outlined potential fixes that could address these issues in my vulnerability report:

1. Salad could implement some level of customer screening or verification before allowing them to deploy non-approved containers onto the network. 
2. Salad could implement network-level controls at the containerd level, within the custom WSL image level, or at the HyperV firewall level that block outbound communications to RFC1918 IP space. 

I can confirm that the vulnerability no longer exists - Salad seems to have implemented #2 above in some capacity. I have also heard that process changes (similar to #1) are coming in the future, which would further reduce the risk posed to Chefs on the Salad network. 

# Wrap Up and Final Thoughts

Salad does a great job with platform security given the inherent security risks associated with their business model. They also take the security of their customers and their "Chefs" seriously given the fast rollout of a fix for this vulnerability disclosure. Thanks to everyone on the Salad side for fixing this one up!

There are a couple things I'd like to see from Salad in the future:

1) Tighter controls on what customers can run on the platform. This was called out in my vulnerability disclosure, but I still recommend limiting new customers to "pre-approved" container images. I think it's fine if vetted customers can run arbitrary workloads, but right now anyone off the street can run metasploit on a Chef PC. 
2) Malicious container detection. Open source tools like [Dagda](https://github.com/eliasgranderubio/dagda) allow for scanning containers for known malware/hacking-tools/etc. This could be fully automated as part of the container ingestion process. 
3) Malicious activity detection. This would require the most amount of engineering to implement, but the thought is that there should be some detective capabilities within Salad Enterprise Linux (SEL) that would alert on blatantly malicious activity within the container environment. 