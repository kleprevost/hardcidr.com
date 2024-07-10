---
author: ["Kyle LePrevost"]
title: "Social Media Analysis of the 2012 Election"
date: "2012-10-24"
description: "An analysis of the 2012 vice presidential debate from a social media perspective. This was a college project for Clemson's Social Media Listening Center."
summary: "An analysis of the 2012 vice presidential debate from a social media perspective. This was a college project for Clemson's Social Media Listening Center."
tags: ["radian6", "college", "social media"]
categories: ["college projects"]
ShowToc: true
TocOpen: false
---

The big question that comes up every four years always winds up being - how can we best quantify a presidential debate? CNN has tried quantifying public opinion around the debates with it’s “undecided voter” sentiment, rudimentary Twitter analysis, and polling random joes on the street. The problem with these studies is one simple thing - sample size and error. In a world where 1%-2% more votes means victory or defeat, we cannot have small sample sizes that lead to 4-5% error. Large scale social media analysis, thanks to Radian6, seems to solve this issue, and I’ll be breaking down aggregate social data around last night’s VP debate in response to the media’s horrible metrics.

Number of Tweets This metric is a double edged sword for us as researchers.How can we derive any meaning from the aggregate number of tweets about a candidate during the debates? Is it true that there is no such thing as bad press? Regardless, here is the aggregate tweet data from Thursday until Friday, including the spike in tweets about the debate. Our query is simply searching for the names of the candidates, their twitter handles, and common abbreviations of their names.

![](/2012electionchart1.png)

As you can see, Biden clearly led Ryan in the number of people talking about him on Twitter. Let’s run a word frequency cloud on each candidate to get a feel for what people were saying about each candidate.

![](/2012electionchart2.png)

Word clouds can give you a lot of noise, but a couple interesting words pop out. Paul Ryan generated a lot of hits from the hashtag #factsmatter, which was used when the tweet focused on fact checking the candidate. Over on the Biden word cloud you can see some negative words like “rude”, “interrupting”, and “laughing”. It is interesting to note that the opposing candidate in each word cloud is the second largest in frequency, which indicates that there was a lot of direct comparisons made between the candidates.

Topic Analysis What were the hot topics during last night’s debate? Let’s take a look:

![](/2012electionchart3.png)

Defense related topics (Iran, military spending, Libya) were at the forefront of the debate last night, and that is reflected in the social data. The interesting data point here is that the abortion topic only came up for a short time at the end of the debate, but takes second place for most discussed issue over jobs/unemployment and tax reform.

How did this debate affect the presidential candidates when it comes to the issues? The graph on the left shows the aggregate number of posts about each issue over two weeks (starting the day before the VP debate) as they relate to the presidential candidates. The graph on the right shows the same thing over the same span of time, but also includes last night’s debate (as they relate to the presidential candidates, not VPs). This will effectively show us HOW the discourse around the candidates changed as their VPs duked it out last night.

![](/2012electionchart4.png)

![](/2012electionchart5.png)

Notice anything? They are nearly identical. If you look closely abortion and defense got decent sized bumps, but the discourse remained relatively the same. There just wasn’t enough volume to affect the proportion of issues talked about over the last two weeks, but it’ll be interesting to see what happens if we were to compare this graph to the same graph 1-2 weeks from today, to see if the VP debate had any longer term shifts in issues.

Who Won?

This question has always bothered me. There are a million ways to measure who won the debate, and really, is there a binary yes/no answer to this question? Anyway, CNN introduced two hashtags - #BidenWinning and #RyanWinning - which they encouraged their viewers to tweet throughout the night. Well, the results are in!

![](/2012electionchart6.png)

Conclusions

Well, that’s all the data! Interpret it as you will, but it suggests that Biden had a stronger performance comparative to Ryan.

