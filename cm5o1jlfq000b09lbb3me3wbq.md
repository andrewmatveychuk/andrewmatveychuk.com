---
title: "Why 99.99% uptime or SLA is bad"
seoDescription: "Explores why a 99.99% uptime SLA isn't always beneficial, weighing costs versus lost downtime revenue for businesses reliant on computer systems"
datePublished: Tue May 14 2019 04:41:57 GMT+0000 (Coordinated Universal Time)
cuid: cm5o1jlfq000b09lbb3me3wbq
slug: why-99-99-uptime-or-sla-is-bad
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923094/2c681493-b9c3-460a-b8b4-fdbcab655b16.png
tags: system-design, it-operations, service-level-agreements

---

I work a lot with clients who need their computer systems up and running. This is not unusual because today, it is hard to imagine a business that does not rely on computers in its operation: accounting, sales, production, and other departments use computer systems to perform their core functions.

When I discuss with my clients the terms of service agreement, I often hear the following phrases: “We need four (or five) nines SLA…” or better one, “We need maximum uptime…”. However, they are rarely able to clarify their reasons without discussing the following questions:

* How do you understand uptime / SLA?
    
* Do you know what it means for your system?
    
* How much are you willing to pay?
    
* How much money will you lose each hour your system is down?
    
* What is a reasonable uptime?
    

In this post, I will follow through on these questions and show you why a higher SLA is not always better.

## What is uptime?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681919677/25ba0bb9-8669-4dda-a654-63e9109078a3.png align="center")

In the context of computer systems, the terms “uptime,” “SLA,” and “availability” are tightly connected to the concept of [high availability](https://en.wikipedia.org/wiki/High_availability). Often, they are used synonymously, which can cause some misunderstanding.

To speak the same language, let’s start with some definitions. Uptime is a [metric](https://w.wiki/7ecX) of computer system performance that is usually measured in days, hours, minutes, etc., and represents the time when a computer system is operational. The opposite metric is downtime, measured in time units and represents the time when the system is down (non-operational). Availability is almost the same as uptime and is used interchangeably. Still, it is expressed as a percentage of the system’s operational time to the total time it should be functional. All these metrics are usually measured over some time: a year, month, week or day.

An SLA or service level agreement is a document, usually legal, that specifies the aspects of the service provided, in particular, the target value for the system’s availability. Since availability is the most commonly used key aspect of service level agreements, it became a synonym for SLA, which is not always correct.

## What does it really mean in terms of time?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681920484/23ac1f3b-5c4b-4321-99ec-3a8fabf4041f.png align="center")

As we have already figured out, uptime and availability are time-based metrics. You can use a [calculator](https://uptime.is/99.99) or a [conversion cheat sheet](https://royal.pingdom.com/a-handy-uptime-and-downtime-conversion-cheat-sheet/) to get the figures for the specific target levels. For example, 99.99% uptime means a system should be available 99.99% of its designated operational time. The acceptable downtime to reach this goal is 52 minutes per year or 4 minutes per month, assuming we talk about the system required to operate continuously all year round. If we consider 99.9% uptime, the acceptable downtime will be 8h 46m and 43m, respectively, which is ten times less demanding.

Why do I talk about all these hours and minutes? Imagine that you have an e-commerce business that heavily relies on its website to sell products to your customers and make you money. Customers can access your site and probably purchase some products when it runs. When the site is down, you are basically losing money on the orders that could have been processed during this time. In real-world scenarios, a business might depend on dozens of interconnected computer systems with different availability levels, but I use the described example for the sake of simplicity.

It is unlikely that you, as a business owner or CEO, want to lose money. Therefore, you need your website running ideally 100% of the time or somewhat close to that, which could be 99.99%, as in our example. That literally means that you accept downtime for your business of no more than 52 minutes per year or 4 minutes per month.

For now, that is all we need to know about time, so let’s move on to the money side.

## How much does it cost to have 99.99% uptime?

First of all, when you design a new or rebuild an existing computer system, you should always consider its [total cost of ownership or TCO](https://en.wikipedia.org/wiki/Total_cost_of_ownership). What does it mean? Simply put, TCO is the money you have to spend on implementing or changing the system plus the cost of supporting that system over time, usually from 3 to 5 years. When discussing the first part of the TCO equation, implementing better SLA or higher uptime will cost you more money. Besides, many inexperienced IT clients often forget or omit the second part, which nowadays may be a much more substantial sum of money due to the subscription nature of many infrastructure, platform and software services required to run your system.

Now, let’s go back to our example with the e-commerce website. Assume that the site’s infrastructure consists of a web app powering the site, some middleware application server to process orders and a backend database to store all the information:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921135/9fc3c496-274a-4f53-bfda-b016dd7a17bc.png align="center")

If you are using Microsoft Azure to host all these components, each mentioned tier will have the following SLAs (all figures are represented up to the time I write this post):

* Azure Web App – [99.95%](https://azure.microsoft.com/en-in/support/legal/sla/app-service/);
    
* Virtual Machine as an application server (with premium disks) – [99.9%](https://azure.microsoft.com/en-in/support/legal/sla/virtual-machines/);
    
* Azure SQL Database – [99.99%](https://azure.microsoft.com/en-in/support/legal/sla/azure-sql-database/).
    

Now the question is, “What is the resulting SLA for this service as a whole?” In our example, if one of the tiers fails, the whole service becomes non-operational for its end users. Therefore, we will calculate the overall SLA as [system availability in series](https://eventhelix.com/RealtimeMantra/FaultHandling/system_reliability_availability.htm):

> Overall SLA = Web Tier SLA \* Middleware Tier SLA \* Backend Tier SLA = 99.95% \* 99.9% \* 99.99% = 99.84% or 99.8% roughly

As you can see, the resulting SLA will be lower than the lowest SLA in a series. 99.8% SLA means that your acceptable downtimes will be [17 and a half hours yearly or almost an hour and a half monthly](https://royal.pingdom.com/a-handy-uptime-and-downtime-conversion-cheat-sheet/). In real life, these figures will be even worse because I don’t consider the probability of human, process and software errors and stick simply to the availability of infrastructure.

You can use the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/) to calculate the cost of the described infrastructure, which will provide you with a 99.8% SLA. For our example, let’s assume it costs $2,000 monthly.

What can be done to increase the overall availability level of our sample infrastructure to the targeted 99.99%? We can either look for options to increase the availability level of each tier or find ways to make the whole infrastructure more resilient. In our case, to reach the target availability level, you can duplicate each tier or basically replicate the entire infrastructure:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921789/f33bb1dc-688b-45cd-a85b-542625f45f7d.png align="center")

By implementing this architecture, you can achieve a service availability of 99.999%, which should satisfy your initial goal of having a service with a 99.99% SLA.  
“Wait! But this means that my monthly infrastructure cost will be twice as high! I will have to pay 4 grand instead of only 2!” you might say. Well, not exactly. You must pay even more for this because you must implement [load balancing](https://azure.microsoft.com/en-us/services/load-balancer/), [traffic management](https://azure.microsoft.com/en-us/services/traffic-manager/) and other failover mechanics to make this infrastructure more robust and autonomous. So, your yearly expenditure on infrastructure can quickly transform from $24,000 to $60,000 or $72,000. To make matters worse, you should also clearly understand that the cost of support for such complex infrastructure will increase because you will need more skilled professionals to operate it.

In practice, to add one more “nine” to your current SLA, you will have to pay three to five times more money than you spend on your current availability level.

## What is the cost of downtime?

Now that we have figured out the cost of achieving “four nines” in our example, it is time to turn back to the downtime figures I mentioned earlier. To simplify the comparison, I have put all the statistics in the following table:

| **SLA** | **99.8%** | **99.99%** |
| --- | --- | --- |
| **Acceptable yearly downtime** | 17h 31m | 0h 52m |
| **Yearly infrastructure cost** | $24,000 | $60,000 |
| **Decrease in downtime** | \- | 16h 39m |
| **Increase in cost** | \- | $36,000 |

I intentionally used time and money values in this table to demonstrate that, in our case, time is indeed money. To have an additional 16 hours and 39 minutes of operational time, you will have to pay $36,000 each year. You will actually be losing this sum of money.

> To be precise, you are losing money in both cases: with 99.8% uptime by not processing orders, while with 99.99% uptime by buying additional time to handle them. Now the question is in which case you lose more.

## Do you really need it?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922463/56d661b5-3cb0-47bb-b5be-2d3932d628cf.png align="center")

At this point, it is high time to ask yourself: “How much money does my system make each hour on average, and how much will it earn me an additional 16 hours and 39 minutes?” You are throwing money away if this sum is less than $36,000. Yes, you have heard it correctly. You are pouring money down the drain. It is not economically feasible to increase the uptime of your system or service unless you are sure it will bring you more money than you spend on such an increase. Moreover, in some cases, it might be more practical, from a financial point of view, to make your uptime worse and cut your operational costs.

I hope that now, after following me through this discussion, you can see that targeting a higher SLA without sound reasoning behind this decision is not as good as it might seem.

Have you ever tried to do such math and find the optimal SLA for your systems? Share your uptime figures in the comments!