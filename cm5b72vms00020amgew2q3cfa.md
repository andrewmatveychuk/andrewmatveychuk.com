---
title: "Incident Management in IT Operations 101 – The Basics. Part 1"
seoDescription: "Learn IT incident management basics, avoid typical mistakes, and distinguish incidents from service requests"
datePublished: Mon May 20 2019 04:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b72vms00020amgew2q3cfa
slug: incident-management-in-it-operations-101-the-basics-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922816/8624a430-381a-441f-8de4-9c6d641b7d8b.png
tags: how-to, incident-management, itil, it-operations

---

With this post, I want to start a [series of publications on dealing with incidents in IT Operations](https://andrewmatveychuk.com/tag/incident-management/) and typical mistakes people make in this process. We will talk about what an incident is, why you should know the difference between an incident and a service request and between an incident and a problem, why speed matters and why it is terribly bad when you find out that something doesn’t work only from your end users. Also, I will cover such topics as incident logging, routing, ordering them by priority, investigating, resolving and writing postmortems. This content will be useful for junior engineers to learn key concepts and for more experienced professionals to understand the connections between incident management and business operations.

> **A word of caution:** this is not a complete course or study material on Incident Management and Service Operation as parts of ITIL framework. There are already plenty of courses on ITIL Foundation on the Internet, including [official ones](https://www.axelos.com/certifications/itil-certifications/itil-foundation-level). If you would like to get an overview of what ITIL is and Incident Management in particular, I can definitely recommend a [four-hour online course at Pluralsight](https://andrewmatveychuk.com/refer/itil-foundation-course).

So, let’s start from the basics.

## I have a theory!

*ITIL and other stuff*

First of all, what is an incident in terms of supporting and running computer systems? [ITIL](https://en.wikipedia.org/wiki/ITIL) defines this in the following way:

> An unplanned interruption to an IT Service or reduction in the quality of an IT service…

To put it simply, an incident in computer systems is a situation in which something has already broken or will break if you don’t take action to prevent it from happening.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681919817/63702ca3-22b0-4bb7-af48-02456eaa6b72.png align="center")

To illustrate this concept, we can use, for example, the server which runs this website. If something happened with the server hardware or operating system and the server became unresponsive, you wouldn’t be able to reach this website and read this blog post. Now you have an incident – the site is down. Some of you might ask why the incident with the website and not the server. Well, it depends on what you value. We might assume, that you, as the user who reads this, rather care about the availability and responsiveness of the site you are visiting than about all its underlying infrastructure and its work. The server failure, in this case, is just a root cause of the incident and your inability to reach the website.

Now, let’s look at the second part of the definition of when something terrible will happen but has not happened yet. To illustrate this, I will use the same server, but, in this case, imagine that its free disk space became critically low due to excessive website log files. The website is working, and everything seems okay yet, but if you don’t take action to free up the server’s disk space in a timely manner, the server will stop working, and the website will go down. Bang! You have an incident.

So, how do these two cases differ? It is an incident severity that makes the difference. In the first case, your site is completely non-operational and inaccessible to its users. No operations can be performed, no orders processed, no sales made, etc. In the second example, your site might be slow or unresponsive but still operational despite degraded performance. There is an issue, too, but it is not as severe as in the previous scenario. However, if you do not take care of this other situation, it might become more critical and lead to the consequences as in the first one.

In some operational frameworks and supporting tools, the definition of an incident may vary. Even the term can be different. For example, instead of “incident,” you might see “alert,” “issue,” or “bug.” In some organizations, engineers may differentiate between incidents, which come from monitoring systems, and support requests submitted by users. Nevertheless, the key idea behind all this is that something is broken and should be fixed.

## Who the hell are you?

*Incident, service request or problem*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681920804/1a774b22-b79b-4e06-9c16-ae9a37b95c79.png align="center")

Now that we have defined an incident, it is the right time to talk about a few more types of work items in IT Operations. You might ask: “Why’s that? Why should we discuss other types if we are discussing incidents now?” Well, unfortunately, when I work with people with no or little experience in supporting enterprise-level infrastructure, I too often observe one specific approach in dealing with incoming requests: people just put them one by one as items on the TODO list. This is a really, really bad idea, and I will explain why.

You might have already heard about or experienced first-hand the process of [triage](https://en.wikipedia.org/wiki/Triage). It is when the priority of patients’ treatment is based on the severity of their conditions. The idea behind the triage process is simple: to save as many lives as possible in the most efficient way. If there are three persons in front of you, one of which is bleeding on a carpet, the second is asking for a prescription, and the third one is complaining about an occasional headache, who are you going to help first? The bleeding person is someone who requires immediate attention. Otherwise, the consequences of inaction might be much worse, an incident in our context. The second man or woman is requesting your service of prescribing medicine and expecting that their service request be fulfilled. The third person is not severely injured right now but is reporting the symptoms of reoccurring accidents of pain over the past few months that might indicate some health problems.

To complete the picture, let’s define what a service request is and what is a problem or problem record. Speaking about the example with patients, if you received an inquiry about your services and the requester is not experiencing any issues with the services already provided for him or her, it is a [service request](https://wiki.en.it-processmaps.com/index.php/Request_Fulfilment#Service_Request). If you observe that some issues, basically incidents, happen again and again, there might be some reason, or a [problem](https://wiki.en.it-processmaps.com/index.php/Problem_Management#Problem), in your infrastructure or configuration that requires investigation and working out. These are not official definitions but simplified examples so you can get the main idea about what these types of items are.

I am not going into too many details on service requests and problem records here, but it is essential to distinguish them from incidents and process them according to their own rules.

## FIFO, LIFO or who screams louder

*Incident priority*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921524/848cdac0-57e2-4fa5-bcbc-4c4f1a5bf572.png align="center")

So, as we already figured out, when handling incidents, it is important to know who you will treat first or how to triage the incoming flow. In our example with three patients, it was obvious, but what if you have a dozen people and all of them are injured in one way or another? You need some rules or system of rules in place that will allow your personnel to evaluate quickly the severity of injuries and the condition of patients, and to sort them in different groups or order them in a queue for treatment.

For example, you might take care of them one by one in chronological order as they arrive at the hospital. Alternatively, you might reverse that order and work with patients from the end of the queue first. In the same way, you work with a stack of papers on your desk. Or, you can turn your attention to a person who yells out, “Help me!” louder than anyone else in a room. In the corporate world, such a person might be a [HiPPO](https://exp-platform.com/hippo/).

As you might already suspect, none of the described methods serve as a good example of saving the maximum number of lives. Moreover, there is no need to reinvent the wheel. [ITIL](https://en.wikipedia.org/wiki/ITIL) already provides an [Incident Prioritization Guideline](https://wiki.en.it-processmaps.com/index.php/Checklist_Incident_Priority) that can be used as a starting point for your specific implementation of an incident triage system.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922180/26bab55e-1225-433e-8124-57ab1568a3fc.png align="center")

> I say “starting point” because ITIL in its nature is a set of recommendations and not a law you must obey strictly.

The idea of incident prioritization is to order them in a queue by their urgency (how serious the issue is) and impact (how many people or business processes are affected) and make this process as fast and straightforward as possible. As with real patients, speed matters and every second counts. The faster you determine the most severe case, the quicker you can start working on it and will have higher chances to improve the situation.

In the next part, we will consider how to handle incidents, who is responsible for their resolution and how they relate to SLAs. Stay tuned and ask your questions in the comments! 👇