---
title: "How to find unused Azure resources"
seoDescription: "Identify, assess, and manage unused Azure resources to save costs, improve security, and enhance operational efficiency"
datePublished: Fri Jun 23 2023 12:30:24 GMT+0000 (Coordinated Universal Time)
cuid: cm5b5oz9h000a09mq856sgrwc
slug: how-to-find-unused-or-orphan-resources-in-azure
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925848/76e00335-a263-41bb-b585-1bb42355dbc3.png
tags: azure, how-to, finops, it-operations

---

This is probably one of the top 10 questions I hear from customers running their workloads in Azure. The question’s intent might vary, but its ultimate goal is the same — to remove unused resources. Let’s explore why we keep facing that question and why most people struggle to answer it.

## Why find unused Azure resources

If no negative factors were associated with unused cloud resources, nobody would probably care about their existence and what to do with them. In other words, when you want to find such resources in your cloud environment, you usually pursue a specific goal. It can be:

* save on the associated resource costs (aka cost optimization),
    
* reduce a possible attack surface on your infrastructure (aka security hardening),
    
* reduce management overhead associated with supporting resources (aka operational excellence),
    
* scale down overprovisioned resources (aka cost optimization),
    
* remove ambiguity when deploying or updating your applications so no incorrect resources are used in the process (reduce errors) and others.
    

Identifying your primary intent(s) for your search will help you to analyze the question from the proper perspective. Why I’m saying that? Because the answer to the question of unused resources depends on your understanding of it. Let me illustrate this with an example.

Have you ever considered why we get unused resources in the cloud? Large enterprises might run hundreds of applications and have thousands of infrastructure components. Infrastructure changes happen constantly: applications get updated, new services are deployed, some are decommissioned, internal processes change, reorgs happen, etc. Changes in one area might not be well-coordinated with others, causing gaps in the existing processes and creating misconfigurations. The number of those misconfigurations, which included unused resources, might be estimated in hundreds and thousands of individual cloud services. The time and resources you can spend processing so much data are usually limited, so focusing on the most important parts makes perfect sense. You should identify and focus on the primary cost drivers to optimize your cloud costs. If your focus is security, you likely want to address the most critical security issues caused by unused resources (an Azure VM with public IP and open management ports is a more significant threat to your environment than an empty resource group or unattached disk).

Now that you know what specific issues you want to address by finding ‘unused’ resources in your Azure environment, it’s time to do the assessment.

## Assessment tools to identify orphan resources in Azure

> Disclaimer. As I mentioned earlier, the exact definition of resource usedness depends on the issue you are trying to solve by looking for those resources. Most of the existing solutions for assessing your Azure environment for unused resources make their suggestions based primarily on the current resource configuration and look for the most obvious cases, like dangling resources that have no practical use in that state, e.g., unattached NSGs, or compute resource not associated with any workloads, e.g., App Service Plans running no apps. However, even those seemingly apparent cases of unused resources might be delusive. For example, an unattached disk might contain some data that is yet to be processed, an empty resource group might serve as a deployment boundary with preconfigured permissions for disposable deployments, etc. So, please remember to check with resource owners or responsible teams on such resources before planning them for removal.

Now, let’s explore some starter tools that you can use to fund unused resources.

[The Azure Orphan Resources Workbook](https://github.com/dolevshor/azure-orphan-resources) is probably the best solution for quickly assessing your Azure subscriptions for unused resources. Under the hood, that workbook uses Azure Resource Graph queries to pull the data about resources in scope and can provide you with a solid starting point for further investigation. It’s a solid choice when you must analyze a new environment and identify the most problematic areas to focus on.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922731/35b06011-232d-4023-9be4-24cf89ea4412.png align="center")

[Continuous Cloud Optimization Insights](https://github.com/Azure/CCOInsights) is a solution based on Power BI. It pulls data from different Azure services and combines it to give you more informative reports on different aspects of your cloud environments. Rather than focusing exclusively on tracking unused resources, it can help you to aggregate recommendations from different systems like Azure Advisor, Microsoft Defender for Cloud, Azure Monitor and so on, providing you with more meaningful insights into the actual resource usage.

Although [Azure Advisor](https://learn.microsoft.com/en-us/azure/advisor/advisor-overview) might not be so powerful in generating recommendations about unused resources as more specialized tools, it can still give you a list of top cloud waste sources and recommendations on addressing them, especially regarding underused resource capacity and possible security risks. It’s free, part of the Azure portal, and doesn’t require any configuration. Its recommendations are structured into several categories that explicitly define your search end goal.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923570/4c07cbe2-5481-4988-8699-0c48bdfe0fa5.png align="center")

[Azure Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/) is a low-level tool for analyzing the configuration of Azure resources. Using [Kusto](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/) (aka KQL) queries, you can quickly find unattached disks, NSGs not associated with any network or interface, etc. If the Azure Orphan Resources Workbook provides a good summary, the resource graph allows you to dig into details and tune your search.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924166/ee1a5a93-d88b-4fe0-94c2-806ef7fa915c.png align="center")

[Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/overview) might not be an obvious choice for exploring unused resources, but it is the number one place to analyze your Azure resource utilization. There are many cases when Azure resource configuration tells you nothing about whether it is actually used or not. However, you can make some assumptions based on resource metrics. For example, you can check how many requests were served by an App Service, the resource utilization for a specific App Service plan, or how many reads and writes occurred from a Storage account over the last time. When you see low resource usage, you should add such resources to your analysis list to confirm their actuality and usefulness.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924770/59e7a581-dff6-4241-8fdc-90222b61b66c.png align="center")

Still, using all those instruments to analyze your infrastructure for unused resources is a reactive action. What can we do to prevent or, better yet, minimize such a waste of cloud resources?

## How to prevent cloud waste

To see how we can prevent the waste of cloud resources from happening, we first need to understand why it occurs.

Have you ever wondered why we store unused resources in the cloud? Unfortunately, there is no easy answer to that question.

Probably, the number one reason for cloud resource waste is the lack of lifecycle management for cloud resources, similar to [Application lifecycle management (ALM)](https://en.wikipedia.org/wiki/Application_lifecycle_management). It’s usually closely connected to absent or underperforming [CMDB](https://en.wikipedia.org/wiki/Configuration_management_database) for cloud resources when you cannot identify resource owners, what applications or services they are related to, and if they are still in use. If you don’t have a CMDB for your Azure resources, I encourage you to check [my series on running CMDB for Azure](https://andrewmatveychuk.com/tag/cmdb/) to understand why keeping your infrastructure neat and clean is essential.

In the second place, I would put the [lack of resource performance monitoring](https://andrewmatveychuk.com/top-5-mistakes-in-it-monitoring/) and established resource [capacity management](https://en.wikipedia.org/wiki/Capacity_management) processes. When you don’t monitor resource utilization and have no insights into how they are used, you cannot make any deliberate decisions about scaling your resources up or down to use them according to current demand. For example, your website might be in use and running just fine on a high-performant App Service Premium SKU. However, it utilizes just a tiny fraction of your plan capacity and can perform equally well on a far less powerful (and less expensive) App Service plan tier. The same applies to storage resources when your actual demand in throughput and data usage pattern can be satisfied using less performance storage tiers (e.g., HDD instead of SDD) and appropriate storage type (Hot, Cool, Archive, etc.). Although autoscaling and consumption-based (serverless) resources partially solve the issue of resource overprovisioning, those cloud-native patterns should be explicitly used when configuring your cloud resources.

The third and not-so-explicit reason for cloud waste is the lack of established security controls for cloud resources and apps running on them. Despite those two not being directly connected, there is often a clear correlation between how secure an environment is and how many wasted resources are in it. Let me explain.

Many security practices involve periodic assessments and automatic scanning for common security vulnerabilities, resource misconfiguration, possible unauthorized access, or data leaks. The more resources you have, the more actions you might be tasked to take to mitigate discovered security threats. Naturally, you would be eager to reduce the number of resources (and apps) in your responsibility to decrease the amount of work you need to perform during each security review. Besides, regular security assessments serve as a trigger to “delete those resources Bob/Dave/Sarah, etc., deployed to do some tests N-years ago.”

As you might already see from that non-inclusive list of reasons for cloud waste, completely preventing it might be quite problematic. Instead, I would suggest focusing on minimizing the negative impact of unused resources during both the design and operation phases of your cloud infrastructure management. When designing your cloud application, you should leverage cloud-native design patterns and prefer solutions that support autoscaling and consumption-based pricing. Whereas operating existing resources requires many infrastructure support processes to be present and executed efficiently. To name a few:

* Configuration Management (CMDB)
    
* Monitoring (performance, events, infrastructure specific and application-tailored)
    
* Capacity Management
    
* Security/Vulnerability scans
    

Also, other practices such as Continues Deployment, Infrastructure as Code, Disposable Environments, Immutable Infrastructure, Application Performance Management, etc., can help you minimize your cloud waste and bring the management of your cloud resources to a new level. Nevertheless, before deciding on what prevention measures to take, don’t forget to clarify your definition of resource usefulness and assess your environment for the largest gaps in effective resource utilization.

What is your experience in eliminating cloud waste and tracking unused cloud resources in Azure or another cloud? Please share your thoughts and suggestions in the comments 👇