---
title: "Notes on “The Phoenix Project” by Gene Kim and others (Book Review)"
seoTitle: "The Phoenix Project" Book Review Notes"
seoDescription: "Review of "The Phoenix Project" exploring IT challenges, DevOps insights, and solutions for system administrators and operations engineers"
datePublished: Wed Sep 04 2019 04:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b3nius000009js2a539pnc
slug: notes-on-the-phoenix-project
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681927016/3a77b95b-8da3-449f-b602-97447efec4ec.png
tags: bookreview

---

“[The Phoenix Project](https://andrewmatveychuk.com/refer/the-phoenix-project)” is one of these rare books you read and find yourself thinking: “Hey man, it is the same situation I am/was!” and these “Aha!” moments follow you all the way to the end of reading. When I finished the book, my biggest impression was: “I should have read this book five years earlier! I spent the last few years wearing Bill Palmer’s shoes and dealing with the same problems in IT operation.”

Even though the book is fiction, it illustrates the most common challenges in IT more vividly than any other guide or technical writing on DevOps practices. To be honest, I listened to the audio version of “The Phoenix Project” more than a year ago but decided to share my thoughts only now because since “reading” this book, I’ve read or listened to dozens of other books on DevOps, and none could compare to it.

## Firefighting

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922785/4bbb01e2-afb2-4af4-979f-cf76aeae7337.png align="center")

One day, you come to work, and then all at once, the trouble starts. Your phone is ringing like crazy. Your coworkers come to your room every few minutes and ask what happened with their email/CRM/HR system or whatever. The whole company operation stops while you, with your teammates, try to find out what caused your production environment to shut down. Sounds familiar to every system administrator or operation engineer, right?

Finally, you find the solution, and all IT systems are back to normal. In the end, you don’t get a word from the big boss; you towel off and return to your daily routines. Unfortunately, in a few days, the major outage in IT systems occurs again. In a while, you realize that no single week passes without some sort of critical incident and you spend most of your workdays fighting them, and the situation becomes only worse.

The described scenario can be observed in different variations in many organizations regardless of their size, maturity or industry. As it often happens, you understand that you are in trouble when the trouble is already big enough to make your work a real pain, and of course, as in all such situations, the cause is usually complex and involves multiple interconnected parts to work with.

So, if you find yourself in a situation as described above, admit that you are in deep shit and be ready that there is no single answer or solution to this problem. You will have to identify the most problematic parts of your infrastructure, processes you follow, and even people you work with, improve them one at a time and then repeat this until you reach the desired baseline of defined target service level objectives.

## Solving change puzzles

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923876/e163e4f6-a9b9-4fcd-8927-33626d1aba0e.png align="center")

Changes in computer system configuration are one of the leading causes of downtime in IT operations. According to some surveys, configuration changes directly or indirectly contribute to up to 80% of incidents. I would argue about the exact figures, but the point is that it is still a significant share.

From my experience, gained from multiple post-mortem reports and problem investigations, the top three factors in change failures that caused major outages are insufficient communication and information flow among the involved parties, poorly prepared or untested changes and a lack of a clearly defined rollback plan.

The scenario in which one team changes the configuration of their part of a system and, at the same time, the second team makes changes in another subsystem became a classic of IT Operations. If you are lucky, you will immediately observe some errors or get incidents from your monitoring service and begin the investigation. In the worst case, you just put a time bomb in your system that will explode by [Murphy’s law](https://en.wikipedia.org/wiki/Murphy%27s_law) in the most inappropriate time. When this happens, the impact and the cost of that incident will be more significant than if you discovered the issue just in the process or right after finishing the change.

The first reaction I usually observe is to introduce more quality checks, verifications and multi-level approvals. This inevitably leads to a creepy change management process with dreadful four-page forms to be filled in before each change and change schedule when you must plan your maintenance windows weeks or even months ahead. Consequently, your responsiveness to business needs and new challenges becomes as slow as a snail.

On the contrary, if you make your change registration and approval routines as fast and straightforward as possible, you will encourage your team to follow the process and not use shortcuts. In addition, if you pre-approve standard changes and select only those that affect your most critical systems, you will have much fewer change requests for approval and will be able to dedicate more time to reviewing them.

## Bus factor

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924501/1c93b3eb-5d93-400c-93be-05a944f98805.png align="center")

Brent, one of the characters in “[The Phoenix Project](https://andrewmatveychuk.com/refer/the-phoenix-project),” is a lead system engineer in Parts Unlimited and perfectly illustrates the [bus factor](https://en.wikipedia.org/wiki/Bus_factor). When you have only one person in your team who everyone else depends on in their work, you are in great danger, and I’m not kidding.

Imagine your one and only qualified teammate who works on a critical part of your project or supports a computer system is unavailable for some reason: sick leave, family issues, unexpected resignation, etc. As a result, you will miss project deadlines, or your whole production pipeline will be stuck and won’t produce any value for the business unless you have somebody who knows how to do the same job as the missing employee did. Many novice managers don’t understand the risks of having a [single point of failure](https://en.wikipedia.org/wiki/Single_point_of_failure) to a full extent until that failure hurts them.

The basics you can do to save yourself from such trouble is to:

a) encourage knowledge sharing among your teammates;  
b) have your colleagues work in pairs and compile a list of these pairs so everybody knows who can cover John, Emma or Bob if they are missing.

## The slowest member

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925545/d2b2295a-1c5e-4837-be80-8e99cc60e9a3.png align="center")

Even if everything is not as bad as if this crucial team member were missing, you still have better evaluate your process(s) as a whole and find the parts that cost you the most time or work effort to perform. You might be surprised, but usually, the most time-consuming and resource-consuming parts are those that are performed manually, i.e., depending on the availability and throughput of specific employees.

If you have five teams of developers that depend on a single release engineer to deploy their software, congratulations. However, you have a constraint. Each time the release engineer is busy with one team’s deployment, the other team has to wait for him or her.

The same is true for operational teams, which have limited capacity and must prepare the infrastructure for new deployments based on requests from multiple developers. If your Ops team has to set up new environments manually, you will sooner or later find your deployment requests queued and waiting for processing.

Both examples mentioned above are the illustration of the [theory of constraints](https://en.wikipedia.org/wiki/Theory_of_constraints). If I were a project or operational manager, I would definitely dedicate my time to studying the TOC and learning how to improve work performance around constraints. Why so? Because any other improvements to the processes will be just a waste of time and money. What’s the point in releasing new versions of software daily if you can deploy them only once a month? 29 of 30 releases have no real value to the users when they work with only one, presumably the latest, release.

## Injecting security

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681926298/9917b0d1-faa7-4c0a-aa37-a97d21561dfd.png align="center")

If you have ever participated in meetings where you present a new cool idea to implement and some colleagues immediately object that “It is insecure!” you are probably familiar with the full spectrum of unpleasant feelings. You might wonder, why does that gay or gal constantly put roadblocks on your initiatives? The truth is simple: they are just doing their job – to keep systems secure and to prevent potential security incidents and data breaches, which, by the way, might be very costly for any business.

The same is true in teamwork when the whole team had worked hard towards completing an important project, and at the approval meeting got rejected “because the solution has critical security flaws.” It is well known that the further you move in your project without assessing the security issues, the more expensive it will be to resolve them in later stages. However, I often witnessed heated arguments between project and security teams, after which project people tend to reduce their interaction with the security ones to the minimum, e.g., sending a request for security approval when a product is already built and waiting for deployment. Inevitably, this approach only makes things worse.

So, it will be wise to review the approach for collaboration with security engineers and involve them in a project from the very beginning. Little time spent on project start for the security baseline of an application or system will save much of your effort upon its release.

I definitely recommend “[The Phoenix Project](https://andrewmatveychuk.com/refer/the-phoenix-project)” to every team leader and professional, regardless of his or her experience in IT. The narrative style of this book helps to understand the key ideas of DevOps in the context of organizations and people involved in transformation much better than any other formal or business-style guide on DevOps practices. If you would like to learn more about the practical aspects of DevOps methodology, check out “[The DevOps Handbook](https://andrewmatveychuk.com/refer/the-devops-handbook)” by the same authors.