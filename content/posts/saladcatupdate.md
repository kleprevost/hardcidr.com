---
author: ["Kyle LePrevost"]
title: "SaladCat Revisited: Pricing Update"
date: "2024-10-07"
description: "An update regarding SaladCat and Salad's new pricing."
summary: "An update regarding SaladCat and Salad's new pricing."
tags: ["cybersecurity", "saladcloud", "salad", "salad.io", "distributed hashcat", "hashtopolis", "vast.ai", "vastai", "hashcat cheap"]
categories: ["cybersecurity"]
ShowToc: true
TocOpen: false
---

_Note: Salad provided me with free credits in order to perform this benchmarking - thanks guys! This post isn't sponsored otherwise._

## Context

I [wrote a blog post](https://hardcidr.com/posts/saladcat/) this past summer about using a startup cloud service called Salad Cloud to perform password cracking using hashcat. I found that you could build some monster password cracking setups fairly cheaply using this service, and showcased that by building a Hashtopolis server with 31 4090 nodes. 

Salad Cloud recently lowered the prices of all of their GPU offerings and have moved to a spot-style "priority" system. You can learn more about their pricing changes [here](https://blog.salad.com/cloud-gpu-pricing/), but the quick summary is that you can pay less money to rent a GPU, but you run the risk of someone paying more to bump your workload off that system. It's a great change, and I figured I'd do a quick analysis of their new pricing model in relation to hashcracking to find out whether the 4090 is still the best value on the platform. 

## The Methodology

I decided to manually benchmark every GPU on Salad Cloud in order to have fresh benchmarking data on the latest hashcat version and accounting for the (gamer) consumer-class hardware on the platform. The team over at Salad provided me with some cloud credits to perform this benchmarking - thanks guys! I tested five different hash types - MD5, SHA2-256, NTLM, bcrypt, and wallet.dat. 

**All of my raw data, pricing, and assumptions about specs are in [this Google Sheet](https://docs.google.com/spreadsheets/d/1-e2kSJd26Ju5icqAiYbfhP-RF6RrKQvtxpfyZcH5Z6U/edit?usp=sharing).** The highlighted/colored columns are the price/performance numbers - represented as (hashrate / hourly cost). All prices assume the lowest priority "Batch" pricing, except one of the 4090 rows which explicitly calls out "High Priority" as a point of reference. 

## MD5 and SHA2-256 Price and Performance

![](/md5.png)

**Top 5 GPUs**
1. 1650 Super (16 GH/s @ $0.023/hr)
2. 1080ti (36 GH/s @ $0.054/hr)
3. 4090 (150 GH/s @ $0.236/hr - BATCH PRICE)
4. 4080 (98.3 GH/s @ $0.178/hr)
5. 4070ti and 3080ti (67 GH/s @ $0.128/hr)

I'm grouping MD5, SHA2-256, and wallet.dat together - they performed very similarly in my benchmarks, so the differences are minimal. 

Surprisingly, the much older 1650 Super and 1080ti cards are extremely good values on Salad Cloud - rivaling the 4090 in terms of hashes/dollar. The only problem with these cards is that there aren't that many of them on Salad and it takes 5-10 of them to match the hashrate of a single 4090. The 4090 at the lowest "Batch" priority is an amazing value, however there aren't many available at the "Batch" price due to high demand. If you plan on running hashcat on Salad, then I recommend setting up Container Groups for these top three GPUs, but understand that it might be difficult to get them due to low supply. 

The 4080, 4070ti, and 3080ti GPUs are the next tiers down in terms of hashes/dollar and are widely available on Salad. They are roughly 15% less performant in terms of hashes/dollar compared to the "tier 1" GPUs above, but should be a lot more stable to run overall due to lower demand and higher supply.

For SHA2-256 you can swap out the 3080ti for the 2070 - oddly the older 2070 performed significantly better in the SHA2-256 benchmark for the price. 

For wallet.dat - the 3060ti is more performant than the 4080 for the price. That was a strange result, but it held up when testing across several different systems. 


## bcrypt Price and Performance

![](/bcrypt.png)

**Top 5 GPUs**
1. 4090 (780 kH/s @ $0.236/hr - BATCH PRICE)
2. 4070ti (773 kH/s @ $0.128/hr)
3. 3080ti (736 kH/s @ $0.128/hr)
4. 3060ti (684 kH/s @ $0.064/hr)
5. 3080 (664 kH/s @ $0.114/hr)

The only surprise here is the 3060ti, which generally isn't a great hashing card on Salad, does surprisingly well for bcrypt. The older 1000-series cards don't hold up well to bcrypt, but the newer 3000-series and 4000-series cards do a great job with breaking bcrypt hashes.

## Conclusion

I'm a big fan of Salad's latest pricing update - it's possible to a 1 TH/s of MD5 password cracking for $1.5/hr, which is insane. This could also be a great way to recover old Bitcoin wallet passwords as well - $40 worth of Salad Cloud credits should be able to crack any 8-character wallet.dat password. 