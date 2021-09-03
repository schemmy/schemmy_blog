---
title: "Sport fans' favor"
date: 2020-09-02T21:32:41-07:00
# draft: true
tags: ["data mining","algorithm"]
---


In the [previous post]({{< ref "../post/research_connections.md" >}}), I played with SimRank algorithm on Arxiv data to show the potential relationship among research areas.

In this post, I would like to apply the same thing on data from a Chinese sport forum, [hupu](https://bbs.hupu.com/). In general, the question is "is there a pattern for sport fans to support different teams? For example, a fan supports Real Madrid will rarely supports Barcelona at the same time". To me, as a deep sport fan, the final question is "For what reasons, a fan becomes a fan for that team"? I'm trying to convince myself based on data mining.

The algorithm used in this post is almost the same as in [here]({{< ref "../post/research_connections.md" >}}), so this will be skiped. The most interesting and time consuming part of this project is data collection. Again, there is no public dataset for me to use directly so I have to play as a spider:) Finally, the following dataset was gathered.

* 50 team communities on [hupu](https://bbs.hupu.com/), 30 NBA teams and 20 Euro football clubs.
* 98k posts, which have 1.2 million comments in total in 2017.
* These posts and comments belong to 53k users.

The interest concentrates on "how communities are linked by users", i.e., a user who is a fan of Cristiano Ronaldo may be active in both communities of Real Madrid and Juventus.

The results I obtained is shown in the following visualization. Thanks to [Gephi](https://gephi.org/) again. Some of the team names are covered, so feel free to download and play with the .gephi file [here](/img_post/sportfan.gephi).

![image](/img_post/sportfan.png)

Some further thoughts and potential TODOs:
1. Basketball fans are basketball fans, football fans are football fans. The connections between these two sports are not strong.
2. The data I collected spans only a year, which might be too short to consider "fans transition". I wish to have data in a longer term.
3. Only consider communities for each team. There are also communities that are open to all fans to discuss, from which we could learnt more. However, this will bring the problem to the next level difficulty, since NLP will be necessary to analyze which team they are talking about, and what are their positions in each post and comment.