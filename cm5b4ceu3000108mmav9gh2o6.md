---
title: "Notes on “Extreme Ownership” by Jocko Willink and Leif Babin (Book Review) - Part 1"
seoTitle: "Extreme Ownership" Insights: Book Review Part 1"
seoDescription: "Explore key leadership and management principles from "Extreme Ownership" by Jocko Willink and Leif Babin, applied to IT operations"
datePublished: Mon Apr 22 2019 08:33:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b4ceu3000108mmav9gh2o6
slug: notes-on-extreme-ownership-by-jocko-willink-and-leif-babin-book-review-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737045922468/14d6ff3c-a1e9-4865-8a64-273df0d86134.png
tags: bookreview

---

Recently, I finished reading or actually listening to [a solid book](https://andrewmatveychuk.com/refer/extreme-ownership) about the key principles that every leader, manager and man should apply in their lives. The authors, weathered veterans of [military campaigns in Iraq](https://en.wikipedia.org/wiki/List_of_coalition_military_operations_of_the_Iraq_War), wrote very thoughtful nonfiction about the foundation of leadership in a much broader sense than one could imagine.

By the time I read “[Extreme Ownership](https://andrewmatveychuk.com/refer/extreme-ownership)”, I was already familiar with some of the principles described in the book, but the situations they were applied to were somewhat new and unusual for me. The most extreme conditions you could imagine, basically a war, emphasized for me how important moral rules are in our everyday lives.

Before exploring the ideas, I want to warn you that this is not a book summary but rather my interpretation of how I see the book’s concepts applied to my background in IT operations and people management. So, if you want to know the actual book context, I strongly encourage you to [read the book](https://andrewmatveychuk.com/refer/extreme-ownership) itself.

So, let’s start with the first principle of extreme ownership.

## Blame yourself first

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922765/e6b5a02e-9603-4730-8fb2-53f9bb4dce4c.png align="center")

If something goes wrong and not as we expected, we tend to look for a scapegoat and victimize ourselves. I must honestly admit that I am not different and am also prone to this mindset. Every time I am challenged with something, I don’t know how to deal with it yet. Too often, I argued to myself that I couldn’t accomplish something at work because of external reasons or another person’s actions or inaction. It costs me substantial time and recurring mistakes to make to recognize the flaws of such an approach. Trust me; this experience wasn’t pleasant.

The first step in dealing with the described way of thinking is knowing that at work, you are paid for solving problems and not for looking for reasons why you cannot deal with them. The goal of every business is profits, not excuses. Period.

If a support request wasn’t fulfilled in time, a website was down, or a server was slow and unresponsive, ask yourself first: What have you done to prevent this or to fix it in the most appropriate way? If your team is underperforming or receiving complaints about their work, question yourself as a team leader: What steps did you take or didn’t take to improve the situation?

If something in your team goes wrong, you are the only one responsible — not your teammates or other colleagues. It is your responsibility to get the work done, either by yourself or by directing the workforce you are granted towards a goal. When I work with team leaders or managers with no or little experience in people management, this is the most important attitude I look for in them.

From my experience, owning the problem is the first mindset challenge you face when you start working. If you already have this attitude in your work or personal life, don’t grudge the time it takes to explain it to your subordinates.

## Cover move

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923718/8e5c5f80-b60f-442a-a0fd-f94f88b04fe0.png align="center")

Despite its origin in [military tactics](https://en.wikipedia.org/wiki/Fire_and_movement), this principle is widely applicable in other areas. Regarding [IT Operations](https://en.wikipedia.org/wiki/Information_technology_operations), this concept can be viewed as having a backup solution while planning changes to IT infrastructure. During my career, I witnessed too many outages and lasting service downtimes due to poorly planned changes.

Many IT services are required to be available 24/7 with minimal or no downtime. This requirement is just a fact of life in today’s competitive world, where businesses want to increase their profits by servicing customers non-stop.

Some might wonder, why care about covering your action if we already planned everything in detail? The truth, however, is that most computer systems and software are so complex today that it is just not economically feasible to forecast every possible scenario. In business, you have to find a balance between a service value and its operational cost.

As a change manager, during reviews, I asked the following questions countless times:

* What are we going to do if a change is not going as planned?
    
* What if the change outcome is not as desired?
    
* Do we have a backup solution if a system stops working?
    
* What is the rollback plan if we need to recover a system to a previous state?
    

These what-if questions are the most typical ones, and the sequence can be continued.

Due to the above, a common change plan almost always carries some risks. The mastery of change management is to identify those risks, perform a risk assessment, and plan a cover move if a negative outcome of these risks is observed. In IT [Change Management](https://w.wiki/Chy$), this cover can be in the form of having a rollback procedure and/or having a backup person or team who is ready to assist you while you are proceeding with a change.

## Keep it simple

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924305/fe08c765-fd6a-4238-9082-f4947bbbfacb.png align="left")

As human beings, when faced with a problem to solve, we tend to choose the solution that requires the least effort to implement it. This behavior is incorporated into our nature as a means of having better chances for survival. From a biological perspective, intellectual work requires a significant amount of energy. If you are challenged physically or mentally with your energy depots depleted, you are less likely to perform as well when your energy level is high. Let’s use IT [Change Management](https://w.wiki/Chy$) again to illustrate this in practice.

Imagine you are assigned a task to change the server configuration. Following the best practices, you planned all steps in advance and composed a detailed action plan, which required a dozen pages to document. You specified who, when and how should perform each part of the change. The plan is so complex that only to read it your colleagues have to spend a couple of hours. At a predetermined time, you started taking actions according to the plan. Suddenly, after an hour or so of focused work, you got a message from your peer that something went wrong and he or she could not proceed further. You still had time to the [point of no return](https://en.wikipedia.org/wiki/Point_of_no_return), and you got everybody on a call to investigate the issue. During the discussion, you discovered that you had missed some critical parts while planning the change, and nobody noticed that. To make the situation worse, one of the junior colleagues said that he or she probably misunderstood some parts and performed some actions differently than was intended. Having all this information in hand, now you see that you don’t have enough time to adjust the plan because it will take too long to verify it and will be too risky to move forward without careful consideration. You have to make a call and roll back all changes. The maintenance window is missed, and the time of all team members is wasted.

Of course, some might argue that there could be other reasons, such as poor planning or misunderstanding, in the described case. Don’t get me wrong. I intentionally simplified the story and omitted the reasons that might have led to similar negative results to illustrate the primary cause of most fails — excessive complexity, which almost always is the root of all IT evil.

Years of evolution proved that humans are much more likely to succeed in a task when they have a shorter set of instructions or a simpler plan in mind. The more complex the plan, the higher the risk of failure you get. From my experience, this dependency between complexity and risks is rather [exponential](https://en.wikipedia.org/wiki/Exponential_function) than linear. So, keep this simple idea in mind next time you are planning to do something on your own are as a team member.

If you want to know about four more principles of [“Extreme Ownership,”](https://andrewmatveychuk.com/refer/extreme-ownership) stay tuned and check [Part 2](https://andrewmatveychuk.com/notes-on-extreme-ownership-by-jocko-willink-and-leif-babin-book-review-part-2) of my notes.