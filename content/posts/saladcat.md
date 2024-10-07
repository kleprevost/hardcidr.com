---
author: ["Kyle LePrevost"]
title: "SaladCat: Distributed Password Cracking on the Cheap Using Salad Cloud"
date: "2024-07-10"
description: "I used Salad Cloud to build a distributed hashcat cracking setup for password cracking."
summary: "I built a ridiculous distributed hashcat system that ran 31 Nvidia 4090 GPUs simultaneously."
tags: ["cybersecurity", "saladcloud", "salad", "salad.io", "distributed hashcat", "hashtopolis", "vast.ai", "vastai", "hashcat cheap"]
categories: ["cybersecurity"]
ShowToc: true
TocOpen: false
---

_Note: Note: I have no affiliation with Salad Cloud and this post isn't sponsored._

## Context

I recently discovered a small startup cloud service called [Salad Cloud](https://salad.com/) which I've been playing around with for the last month. It caught my interest as a cybersecurity researcher for a variety of reasons and I've had quite a bit of fun tinkering with this platform. Salad's core business model is simple: gamers can "rent out" their gaming PCs when they are "AFK" to Salad Cloud customers who want to run AI workloads on the cheap. "Gamers" can earn a bit of spending money, companies get easy access to consumer GPUs, and Salad takes a 10%-20% cut for making the whole thing happen. The technology stack that makes this whole thing work is fascinating as well, but I'll save that for another blog post.  

Most Salad Cloud customers are using the platform for AI use cases - Stable Diffusion-based image generation (CivitAI) and Whisper-based audio transcription seem to be their bread and butter currently. That's cool and all, but as I was playing around with Salad I couldn't shake one intrusive thought: "I bet you could build a ridiculously large password cracking setup on here." Despite not even having a need for a large hashcat deployment at the moment, I succumbed to these intrusive thoughts and built out a distributed hashcat deployment that works on Salad Cloud. And to be honest, it worked a bit too well.

![Picture of 31 RTX4090s cracking a demo MD5](/hashcat1.png)

![Picture of 31 RTX4090s cracking a demo MD5](/hashcat3.png)


A quick internet search also showed that this might be the single largest 4090-only hashcat deployment ever built (_and disclosed publicly_). The next largest deployment I could find was [Kevin Mitnick's password cracking setup](https://hothardware.com/news/famed-hacker-unveils-password-cracker-dozens-rtx-4090s) which uses 24 4090s in a single rig. If anyone knows of larger hashcat deployments using 4090s, let me know!

I ran my 30+ 4090s against a pretty basic MD5 hash for about an hour - the whole thing cost me $10 before I shut it down. 

## Technical Implementation and Code

**[GitHub Repo](https://github.com/kleprevost/saladcat)**

**[DockerHub](https://hub.docker.com/r/kleprevost/hashtopolis-hashcat-salad)**

Salad Cloud is a PaaS similar to Azure App Services or AWS ECS - you give it a container workload and it runs on a random "gamer" PC somewhere in the world. The catch is that these nodes need to be treated like spot instances since "gamers" can stop "chopping" at any time, which kills our container immediately. I decided to go with Hashtopolis as the core technology for this implementation since it pretty effectively manages distributed hashcat workloads and doesn't get too upset if a node suddenly dies. 

_Sidenote: I'm using "gamer" in parentheses because the typical "Salad Chef" who rents their compute power to Salad has been moving away from just the gaming community. Former cryptominers, for-profit GPU farms, and internet cafes seem to be overtaking the core demographic of the Salad Cloud network._

There were existing implementations of Dockerized hashcat in the wild, but unfortunately none of them worked particularly well. I borrowed extensively from several of those existing containers to create SaladCat, which is a "fire-and-forget" way to get Salad nodes registered in Hashtopolis. 

The [SaladCat GitHub repo](https://github.com/kleprevost/saladcat) goes into detail about how to get everything up and running, but I'll spend some time here talking about how ridiculously easy it is to get it setup. The only prerequisites are a Salad Cloud account (requires $50 deposit, but they give you $25 in "free" credits) and a Hashtopolis server (shoutout to Nikita for [his guide](https://nikita-guliaev.medium.com/clustering-hashcat-with-hashtopolis-for-distributed-cloud-computing-55f964a56804) on setting Hashtopolis up). The following steps will have you up and running with a (relatively inexpensive) password cracker:

1. Generate a voucher in Hashtopolis. Make sure the "Vouchers can be used multiple times and will not be deleted automatically" setting is enabled on the server. 
2. Create a new "container group" on Salad Cloud. If you don't want to do this manually via the portal, then [I have built a script as part of SaladCat](https://github.com/kleprevost/saladcat/blob/main/create_salad_hashcat.py) that uses the API. 
	* The container group should be configured to pull the kleprevost/hashtopolis-hashcat-salad:latest container. 
	* Set two "Environment Variables" on the container group: 1) API_SERVER_URL and 2) VOUCHER. They need to be all uppercase. The values should be the Hashtopolis server URL found under "New Agent" and the voucher you created, respectively. 
	* Set the number of "instances", or hashcracking agents, that you need.
	* I have a breakdown of what GPUs/RAM/CPUs are most cost efficient for hashcat down below if you are waffling between what specs you want to deploy onto. 
3. You can click "Start" to deploy the container group at this point.
4. (Optional) You can configure and run the [update_agents.py](https://github.com/kleprevost/saladcat/blob/main/update_agents.py) script included in the SaladCat repo. This automates the process of marking the agent as "trusted", ensuring it doesn't stop on errors, and finally it can be set to automatically start working on a job. 

That's it! I'm honestly not sure how large Salad's network is, but it only took 30min for me to deploy 30-40 4090 GPUs on the network. 

![Salad Cloud Screenshot](/hashcat4.png)
![Salad Cloud Screenshot](/hashcat5.png)



A couple known issues if you are looking to replicate (or beat) my current 4090 monstrosity:

1) Salad Cloud limits new customers to a max of 3 container groups with 3 instances each. I contacted their support (via the Portal) and got that limit bumped up to 10 container groups with 10 instances each within 24hrs of submitting the request.
2) There is an outstanding issue where individual hashcat agents will fail the speedtest. It happens 10%-15% of the time, but I'm sure it's an issue with my deployment. I'm not an expert on hashcat/hashtopolis, so it's probably something silly. 
3) The agent name in Hashtopolis seems to be the container group ID. This makes booting or "re-allocating" dead agents difficult. I'm sure the Dockerfile or agent config can be configured to pull the SALAD_MACHINE_ID from the env to set the agent name correctly.
4) Right now Salad can only guarantee up to 50gb of free space on the agent excluding the container size. That being said I believe you can use as much free space as available on the host PC. This might make deploying very large wordlists difficult or inconsistent as every host will have extremely variable storage capacities. 

## What specs are the most cost-efficient?


**10/07/24 EDIT: I recently did a more in depth analysis of Salad's pricing in relation to hashcat performance. You can find that [here](https://hardcidr.com/posts/saladcatupdate/).** 

TLDR: the 4090 is the best bang-for-your-buck. You'll want to give it enough RAM and CPU cores, but even then it's the king of cost efficiency by a mile on the Salad platform. 

The obvious cost-driver in a setup like this will be the GPU - the bigger and more modern GPUs will crack faster, but are also more expensive. What about the other hardware like CPU and RAM? There is a [long discussion over on the hashcat forums](https://hashcat.net/forum/thread-11041.html) that talks about the CPU/RAM specs required to get the most out of your GPU. The consensus seems to be: provision at least as much RAM as there is GPU VRAM (so a 24gb 4090 should have at least 24gb of system RAM) and at least 1 CPU core per GPU. 

Salad only supports one GPU per host node, so the bare-minimum cost setup will be: 1 vCPU and RAM=VRAM. The table below shows off the cost efficiency of each GPU at Salad's retail pricing. 

| GPU          | Cost/hr  | MD5 Efficiency (billion hashes/$) | SHA256 Efficiency (billion hashes/$) | NTLM Efficiency (billion hashes/$) | bcrypt Efficiency (thousand hashes/$) |
|--------------|----------|----------------------------------|--------------------------------------|-------------------------------------|---------------------------------------|
| 4090         | $0.328   | 500.30                           | 67.00                                | 879.57                              | 561.59                                |
| 4080         | $0.300   | 327.54                           | 43.59                                | 577.67                              | 388.33                                |
| 4070ti super | $0.280   | 281.38                           | 32.33                                | 475.71                              | 390.71                                |
| 4070ti       | $0.256   | 264.88                           | 35.37                                | 546.88                              | 427.34                                |
| 4070         | $0.236   | 233.89                           | 33.17                                | 389.62                              | 97.69                                 |
| 3090         | $0.278   | 286.87                           | 38.58                                | 497.48                              | 415.83                                |
| 3060ti       | $0.092   | 348.49                           | 50.87                                | 635.78                              | 476.40                                |
| 2080         | $0.092   | 403.10                           | 58.27                                | 575.59                              | 201.90                                |
| 1080         | $0.052   | 479.67                           | 55.10                                | 804.33                              | 251.81                                |

## What about Vast.ai?

I've been asked this question frequently as I've been socializing this project with others in the community.

[Vast.AI](https://vast.ai/) is the current "go-to" for renting GPUs for password cracking. It's a decent platform that I've used before, but I now prefer Salad Cloud for the following reasons:

1) Salad is better if you don't have specific hardware requirements. I don't want to filter and select many rentable machines on Vast.ai — I just want to deploy on 4090s and have it work.
2) Somewhat related - scaling up and down is super easy on Salad. And if you get an underperforming node, it's easy to just "re-allocate" and roll the dice on another one. 
3) Salad only charges for the time the container is actively running.
 It's possible to spin up and spin down password cracking jobs with minimal cost impact. Vast.ai has several billing options, but generally there is less flexibility on that platform. 
4) Salad is more expensive than vast.ai when comparing retail prices, but I've seen some aggressive discounting for larger customers (it's possible to glean pricing info through running a node on the "Chef" side). I'd imagine you can get parity with the current vast.ai pricing if you asked nicely. 

## Conclusion

I hope someone finds this blog post useful! This was a fun side project - it has been a long while since I built something outside of work. I'll be doing a couple more blog posts on Salad Cloud from various perspectives - mostly focusing on the technical and cybersecurity angles of the platform. 