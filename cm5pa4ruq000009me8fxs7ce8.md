---
title: "Incident Management in IT Operations 101 – The Basics. Part 2"
seoDescription: "Explore incident lifecycles, workflows, and SLA importance in IT. Understand incident dispatching and effective resolution processes"
datePublished: Mon May 27 2019 04:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5pa4ruq000009me8fxs7ce8
slug: incident-management-in-it-operations-101-the-basics-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737045776998/859bf14f-0893-46df-a8fe-0442c90ddccf.png
tags: incident-management, itil, it-operations

---

In the [first part](https://andrewmatveychuk.com/incident-management-in-it-operations-101-the-basics-part-1) of [this series](https://andrewmatveychuk.com/tag/incident-management), I talked about what an incident is in IT Operation, why you should distinguish them from other types of support tickets and how to put them in order.

In this post, I will continue exploring the basic concepts of [Incident Management](https://w.wiki/Chys).

## If it’s not written down, it never happened

*Incident lifecycle, workflow and states*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681919572/97d34c60-055d-431d-a767-70b982722720.png align="center")

Now that we already know, at least in theory, [what an incident is and how to sort its queue](https://andrewmatveychuk.com/incident-management-in-it-operations-101-the-basics-part-1), it is time to examine the incident lifecycle. Let’s step back for a moment and ask ourselves: “How do we know there is an incident?”

To work on incidents, we need some records of them, a place to note all the information that might be useful for their resolution. Apart from that, it is helpful to share this information with your teammates so they know what is going on and the current status of specific issues. If there is no incident record, tracking any changes and following status updates is hard. So formally, an incident becomes known when the information about it has been logged and a corresponding record has been created. In IT Operations, the proven best practice for incident logging is to use [ITSM](https://w.wiki/Chyt) tools specifically designed for such purpose.

After creating an incident record, support engineers can become aware of some issues and start working toward resolution. After the issue has been resolved, the incident record should be updated with resolution information so other engineers know that this specific issue has already been addressed and there is no need to look into it, at least right now. On the way from point A, an incident happened, to point B, the incident was resolved, the incident record can transition through different states and status updates, e.g.:

* incident acknowledged – a support representative verified and confirmed that the issue indeed exists;
    
* incident assigned to an engineer – incident record has been assigned to a support engineer for investigation and resolution;
    
* work in progress – a support engineer or a team is working on the issue and trying to fix it;
    
* awaiting confirmation – a support representative was not able to verify the issue and is waiting for additional information;
    
* customer pending – a support engineer has made some changes and is waiting for customer feedback to ensure the issue is fixed, etc.
    

The exact incident status names and definitions may vary from system to system and from process implementation to implementation, as well as the workflow described in a [state diagram](https://en.wikipedia.org/wiki/State_diagram) for incident records. Nevertheless, the underlying idea remains the same — to have an explicitly defined workflow for processing all incidents according to the same standard rules.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681920431/caac2b48-ac00-45bb-836d-3d6a3c09c8f8.png align="center")

In most scenarios, a simple workflow that defines only two incident states: “active” and “resolved,” works fine. It is just good enough for 80% of incident management process implementations. If this is insufficient in your specific situation, you should remember that workflows are not carved in stone; therefore, they can and should be tailored to your needs.

## Not my business

*Incident owning, assigning and dispatching*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921091/2a2b578c-f2c0-4e68-a06a-eb3ec6dc8afc.png align="center")

As in my [example with patients at the hospital](https://andrewmatveychuk.com/incident-management-in-it-operations-101-the-basics-part-1), it is good to know who your doctor is. Basically, who is responsible for taking care of a specific incident? If no one is in charge, then there is a high chance that nobody will do something to solve the issue. It has nothing to do with technology but instead with human psychology. It is so easy for many of us to say, “It’s not my business…” and then to take responsibility and accept related risks of making things worse. Of course, someone might say it is a matter of organizational culture, character, and personal attitude. Still, when you have strict SLAs to comply with (more on this later in this post), you do not have the luxury of checking every time whether someone will fix that new dumb issue.

So, how will you ensure every incident has its own “doctor”? You have two options. The first one is to have some person (or a few people) who will dispatch incoming incidents to appropriate support teams or assign them to specific engineers. To do this, the dispatcher should have some rules for routing incidents. The second one is to have some automation system with a particular algorithm to process registered incidents and forward them to relevant technicians. These rules may vary from team to team and from organization to organization, but the key point is to have an algorithm or procedure in place that everybody understands and follows.

From my experience, there is no single best solution for incident dispatching. In some cases, it is possible to have one-to-one relationships between computer systems and corresponding support teams and fully automate incident assignments based on this knowledge. In the other ones, the boundaries among systems and responsibility zones might be blurred, and manual dispatching works best. In some, it is just a straightforward process of assigning all incoming incidents to the [first tier of support (L1)](https://en.wikipedia.org/wiki/Data_center_management#Tech_Support) and later escalating them to a higher tier if needed. Of course, it would be great to automate incident assignments as much as possible, although organizational, experience and process differences might not allow this to happen. So, I advise you to experiment and choose the best for you.

## We will look into your issue as soon as we can…

*Incidents and SLA*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921686/d3e34af8-b504-4ff8-a9f0-4509a514007a.png align="center")

It is crucial that [business operations](https://w.wiki/9yyy)’ supporting processes are predictable. A [Service Level Agreement](https://en.wikipedia.org/wiki/Service-level_agreement), or SLA, is an agreement between a consumer, usually a business, and a supplier, or, in other words, a service provider, which sets such expectations by defining [service-level requirements](https://en.wikipedia.org/wiki/Service-level_agreement). These requirements, for example, may determine that your workplace should be operational [99%](https://andrewmatveychuk.com/why-99-99-uptime-or-sla-is-bad) of business hours. If there are no such expectations, it becomes really hard to run a business. Let me illustrate this point.

For example, as an office employee, you come to work every day to carry out your duties. You are assigned some tasks, so plan your time accordingly to complete these tasks on time. How do you do that? Basically, you assume that you have 8 (or any other number) working hours each working day to work on tasks. You estimate the hours to complete each task, book a time for them on your schedule and start working. But what if each day when you come to work, you don’t know exactly how many hours you will be able to work today? How, in this case, can you make any plans and commitments to complete the tasks in some due time?

Another example is on a business level. When you go to a bank, you fully expect that you will be able to make your payments during the bank’s opening hours. However, what can you do if the bank’s payment processing system is not working and there is no forecast of when it will become operational again? I assume you might decide to transfer your money to another bank that can at least provide you with information on recovery time in case of outages.

To ensure predictability, SLAs can define target [system availability](https://en.wikipedia.org/wiki/High_availability) and [recovery time objectives](https://en.wikipedia.org/wiki/Disaster_recovery#Recovery_time_objective). In other words, there are usually obligations to resolve incidents and recover computer systems from failures within a specified time.

When an incident occurs, and you cannot do what you intended, it is helpful to know when the issue will be resolved so you can adjust your plans.

In the next part of [this series](https://andrewmatveychuk.com/tag/incident-management), I will discuss SLA metrics for measuring process performance and service levels. We will also examine the outcomes of the incident management process and how they can be connected with other activities in IT Operations.

So, stay in touch and ask your questions in the comments!