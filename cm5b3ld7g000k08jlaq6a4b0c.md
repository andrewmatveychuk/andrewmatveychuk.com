---
title: "Notes on “The Unicorn Project” by Gene Kim (Book Review)"
seoTitle: "The Unicorn Project" Book Review Summary"
seoDescription: "Explore Gene Kim's "The Unicorn Project," a novel blending fiction with DevOps insights, highlighting five key ideals for organizational success"
datePublished: Thu Jan 09 2020 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b3ld7g000k08jlaq6a4b0c
slug: notes-on-the-unicorn-project-by-gene-kim-book-review
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737041860750/4310a227-fde9-4492-8550-0ef1447735c9.png
tags: bookreview

---

This winter holiday season, I spent some time [listening to](https://andrewmatveychuk.com/how-i-read-books/) a new book by the author of the famous “[The Phoenix Project](https://andrewmatveychuk.com/notes-on-the-phoenix-project/).” As I really liked that book, a new novel about the DevOps movement was on my reading list long before the release.

“[The Unicorn Project](https://andrewmatveychuk.com/refer/the-unicorn-project)” is fiction, not a technical guide for DevOps practices. So, keep that in mind when looking for your next reading. The story’s events happen during the same period as the events in [“](null)[The Phoenix Project](https://andrewmatveychuk.com/notes-on-the-phoenix-project/)” and complement each other. However, you don’t have to read “[The Phoenix Project](https://andrewmatveychuk.com/notes-on-the-phoenix-project/)” first to understand the setting of the new book.

The book elaborates on “Five Ideals,” or, I would rather say, five ideas that are crucial to creating high-performance organizations. So, let’s discuss them in more detail.

## Locality and Simplicity

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922524/f486ecce-b14d-4d80-a6f0-9eef391338dc.png align="center")

Despite their benefits, the [segregation of duties](https://en.wikipedia.org/wiki/Separation_of_duties) and [functional organizational structure](https://en.wikipedia.org/wiki/Functional_organization) create complex processes and introduce many delays. In extreme cases, such as the Phoenix Project in Parts Unlimited, even simple IT tasks might sometimes require weeks and months to complete.

Modern applications and computer systems usually consist of multiple components: servers, databases, network connections, mobile apps, etc. If a separate team manages each component and you have to create request tickets to ask them for assistance, it becomes like a quest game where you must gather 50+ items in a specific order to complete the task. From the game perspective, it might sound fun for quest lovers, but if you need to submit dozens of requests and get tens of approvals each time to complete simple tasks, it becomes a real nightmare: instead of doing your job, you constantly fight some dumb bureaucracy.

Don’t get me wrong. I’m not advocating against processes. They are necessary to bring order and ensure predictable outcomes of your work. However, when they become Godzillas that require an A1 sheet to draw, it is probably high time to review and simplify them.

In contrast, if your team has most of the required skills to complete 80% of your daily tasks without creating 10+ process diagrams, you might be really impressed by how fast and how much value you can produce in a single day. Also, from the operational point of view, the simpler and smaller a solution is, the faster it works and the less maintenance effort it requires.

## Focus, Flow and Joy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923449/08051d5e-fa1d-42b8-b3cf-caa8e7e49a68.png align="center")

I believe that almost all engineers and developers during their careers have these challenging times when they try to concentrate on an essential task without success. A phone rings, an instant message pops up, a colleague stops by your desk and asks you something – you probably know that feeling when just a moment ago you were close to solving “a puzzle,” and suddenly you get distracted and completely lose your focus. On the contrary, sometimes you can accomplish more in a few quiet hours than in a busy week. So, what’s the trick?

People are bad at multitasking. That’s the truth of life. No magic. Context switching consumes lots of mental energy, requiring 10 to 20 minutes to return to “[The flow](https://andrewmatveychuk.com/refer/flow).” Just imagine that 10 distractions daily can cost you more than 3 hours of productive work. Plus, your energy level is not the same high during the day. So, if you waste your most productive hours on useless meetings or administrative tasks, don’t wonder why you feel exhausted and cannot perform creative work despite having a couple of “free” hours in the afternoon.

I already wrote a few articles about [personal productivity](https://andrewmatveychuk.com/tag/productivity/), so if you want to know more about organizing your time, check them out.

## Improvement of Daily Work

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924114/7429fa0d-0789-4bdb-9c9e-da451b65db68.png align="center")

“Beware the man who says he has 10 years of experience, who has 1 year of experience 10 times.” If you do the same job every day and don’t invest a portion of your efforts into improving it daily, you will either become obsolete due to the automation of your tasks or lose to competitors who seek better ways to do the same job more efficiently.

A few years ago, I was impressed by [the idea of keeping operational work below 50% of your time](https://landing.google.com/sre/sre-book/chapters/eliminating-toil/) described in “[Site Reliability Engineering: How Google Runs Production Systems](https://andrewmatveychuk.com/refer/site-reliability-engineering).” It seems evident that the less toil you have, the more time you can dedicate to improvement. It can be learning new skills, making your system more reliable and responsive, replacing ineffective parts with more productive solutions, decommissioning obsolete systems to free up more productive time, etc. However, it is not so easy to implement that idea in practice because people naturally resist changes.

Apart from that, the widely discussed phenomena of technical debt and the natural tendency of complex systems to entropy contribute to the process deviation from an ideal (most effective at some point) state. Regarding the IT industry, constant changes in hardware, software, processes, and organizations impede our progress toward “a better future.” On the contrary, changes are an unavoidable part of organizational evolution. Something that worked perfectly just yesterday today might become a restrainer due to the changed conditions in a company, on the market, or elsewhere. The truth is simple: you either improve or degrade — no middle ground.

## Psychological Safety

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924768/1510f1ef-7f32-40c8-9689-bd857f544926.png align="center")

I would put this ideal first because fear is one of the most potent natural feelings. It is really unwise to underestimate its devastating impact on individual and team performance. On rare occasions, fear might help to mobilize your resources and achieve extraordinary results, but if this state becomes constant, it will just suck up all your life energy and put you into bed in a better case.

I had experience working in a team where the former manager punished the subordinates on every occasion. Even the most minor mistakes weren’t forgiven. If you were performing as required you lived, if you were falling behind, you got your porting of lashes. The people were unwilling to express any initiative because the best they could expect was “business as usual” without benefits. If something went wrong, you got a fine or pay cut. Sounds motivating? I doubt that. Eventually, that behavior caused the team to stagnate and be unable to perform its duties according to the business requirements.

It took me almost two years to change that team’s perception of mistakes and create an environment of trust. Only when these people believed that they could safely make and correct their errors did the team gain self-sufficiency enough to face the upcoming work issues with confidence. Creating a blameless work environment is, to my mind, the most important work that any manager can do for his or her team. Entrusting your subordinates or teammates to do their best is the prerequisite for long-lasting, high organizational performance.

## Customer Focus

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925451/d70d2645-c342-466f-94d0-12830b401e5a.png align="center")

Understanding your customers, even if they are just other teams in your organization, is the map to your success. Your performance might suck, and you might have huge technical debt, but if you understand what business value you should deliver, it’s like having the map of a pirate’s treasure in hand. You might lack the information about the exact steps required to get to the point, but at least you know where the gold is.

Many average engineers and developers tend to perform their technical work and do what they have been told. On the one hand, focusing on your primary job duties is essential for successful employment. On the other hand, seeing a little bit further and going the extra mile can significantly differentiate you from hundreds and thousands of other tech folks.

The difference between a good and exceptional professional is that the latter can map the flow of business value and understand who exactly contributes to his or her paycheck. According to some research, that difference can be as much as 10x in business impact compared to some John or Jane who just toils from 9 to 5.

## Final notes

Despite my high interest in this book, I wouldn’t call it a masterpiece. This novel has many logical and literary weaknesses, so some book parts should be perceived with a bit of irony. To my mind, too much attention is devoted to the feelings of the main character, Maxine, but the connection between her emotional state and the story is mostly unclear.

Moreover, the motivation of “the rebellion” to change the way “Parts Unlimited” works is also somewhat idealistic and unrealistic at the same time. Judging from the plot, all the rebels just wanted “to do a great job,” “have great feelings about the work they do,” “make the company better,” “feel good about solving complex engineering problems,” and other “blah-blah-blah” mottoes. It looks like there are no other companies or job opportunities in the area to look for, and “the good guys (and gals)” work “to make the company great again.” If I were one of the book’s characters, I would first question whether the cost of such efforts is worth the possible benefits. By benefits, I mean personal goals such as promotion, pay rise, relevant work experience, data for your studies or any other that is appropriate.

The last criticism concerns the synergy level when almost all company employees are good people with only a few “villains.” The bigger the organization, the more forces and parties are involved in the daily politics game. So, in more realistic scenarios, I would expect such drastic transformations to be either more local (see the first ideal) or to involve much stronger leadership intervention to align key political groups, which are natural parts of every large organization. “Life is not always black and white; it’s a million shades of grey.”

Nevertheless, I enjoyed listening to this book and highly recommend it to everyone interested in DevOps culture, organizational performance and self-development. So, go grab your copy and have a good reading!

P.S. If you are looking for other books like that, check out my review of “[The Phoenix Project](https://andrewmatveychuk.com/notes-on-the-phoenix-project).” Also, you can find more practical advice on the principles and ideas of DevOps in the non-fiction “[The DevOps Handbook](https://andrewmatveychuk.com/refer/the-devops-handbook)” and “[Effective DevOps: Building a Culture of Collaboration, Affinity, and Tooling at Scale](https://andrewmatveychuk.com/refer/effective-devops).”