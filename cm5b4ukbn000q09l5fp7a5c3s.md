---
title: "How to monitor Azure Reservations utilization"
seoTitle: "Monitor Azure Reservations Utilization Efficiently"
seoDescription: "Learn how to effectively monitor and manage Azure Reservations utilization to optimize cloud costs and enhance your FinOps practices"
datePublished: Mon Oct 03 2022 13:56:20 GMT+0000 (Coordinated Universal Time)
cuid: cm5b4ukbn000q09l5fp7a5c3s
slug: how-to-monitor-azure-reservations-utilization
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681929702/edba5894-172e-42e0-81ff-9a17142cbd6b.png
tags: azure, finops

---

> **UPDATE**. On May 23, 2023, [Microsoft announced a built-in functionality for alerting about underused Azure Reservations](https://azure.microsoft.com/en-us/updates/rualerts-2/). If you don't need your reporting in a specific format and are okay with a standard notification template from Azure, it's totally fine to use that option in your FinOps processes.

Using Azure Reservations is [one of many techniques](https://andrewmatveychuk.com/practical-use-cases-of-cost-optimization-in-azure) that can optimize your cloud spending in Azure. They are a great option to reduce costs for Azure resources with a long lifespan and predictable utilization. Have a VM, storage, or database that will run ‘permanently’? Buy a reservation for it, set it up to auto-renew upon expiration, and enjoy savings of up to 70% or so. Does it sound too good to be true? Let’s find out.

For small setups with few cloud resources or stable infrastructure with little changes, the easiness of using the reservations is mostly true. However, for large enterprise-scale environments, the reality is a bit different. The larger the environment, the more changes in it you inevitably have. Apart from that, the flexible nature of cloud services, which can be provisioned and decommissioned in a matter of minutes and not weeks or months, provides even more opportunities for engineers to modify their solutions and change their infrastructure requirements. Something intended to run for the next few years can be gone in a few months or replaced with other infrastructure components.

All those changes in a cloud infrastructure create the challenge of managing Azure Reservations efficiently. As the reservations are implicit commitments to pay for a fixed amount of cloud resources over one or three years, it’s essential to utilize those resources fully. Otherwise, you will be paying for resources you actually don’t consume, and all your savings from using those reservations will be diminished.

Luckily for us, Microsoft [allows exchanging existing reservations for new ones](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations). Of course, there are some limitations, but it’s still more optimal to return unused reservation units and buy new reservations for services that you do use. The obvious first question here is how you know what your reservations are underutilized and to what extent.

## Azure Reservations utilization info

Microsoft provides [various options to check reservation utilization after its purchase](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/reservation-utilization). You can check it on the Reservation blade of the Azure Portal, use the corresponding section in the Cost Management + Billing interface, or run some ready-to-use Power BI reports. Also, you can use [Azure PowerShell](https://learn.microsoft.com/en-us/powershell/module/az.reservations/get-azreservation) and [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/reservations/reservation?view=azure-cli-latest#az-reservations-reservation-list) to get some reservation utilization details. Apart from that, you can utilize [the Reservations insight preview feature of the Azure Cost Analysis](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/cost-analysis-built-in-views#review-reservation-resource-utilization).

However, all those options have a common disadvantage — none of them has built-in functionality to automatically notify or alert when utilization for Azure Reservations drops below a certain value. Even though the platform collects data about reservation utilization, and you can check it in the reservation details on the portal, it’s not available as a metric in Azure Monitor.

One of the options to get the reservation utilization info programmatically is to use [Microsoft Cost Management APIs](https://learn.microsoft.com/en-us/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage). On the one hand, with such an API, you are not limited to vendor-provided monitoring options only, and you can use any custom or third-party monitoring solution that can query a REST API, parse a JSON payload, and extract the average utilization percentage value from it to be compared with some threshold.  On the other hand, something that could be a simple Azure Monitor metric alert requires a whole monitoring infrastructure for that and overcomplicates the monitoring setup.

## Monitoring Azure Reservations utilization with Power Automate

As a decrease in Azure Reservations utilization is not something that you usually need to act upon immediately, it might make more sense to build an automated reporting that would provide you with insights about underused reservations on a regular basis related to your FinOps processes. So, by the time you are reviewing your monthly cloud invoices and analyzing your cloud spend, you have the data about what Azure Reservations require assessment and possible exchange or cancellation.

To build such a solution, I decided to try using Power Automate cloud flow. It’s user-friendly and usually available for most Microsoft customers as [part of their Microsoft 365 plans](https://learn.microsoft.com/en-us/power-platform/admin/power-automate-licensing/types#seeded-plans) or as Logic apps in their Azure subscriptions.

> Alternatively, you can implement a similar logic using Automation Runbooks or Azure Functions. My choice of a Power Automate flow (aka Logic app) here is merely a matter of quickly prototyping a solution that can be maintained and modified by people with no specific programming knowledge.

For a start, we can use the mentioned Cost Management APIs to [get the list of all reservations with their usage summary](https://learn.microsoft.com/en-us/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#request-for-reserved-instance-usage-summary), and the HTTP action can help us with that:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922757/44bbbdec-2cee-447b-bc1d-8e8687c0a3d4.png align="center")

Be sure to use the correct enrolment number in the URI, which you can check on the Azure Portal or [the Enterprise Portal](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/ea-portal-get-started) (for EA customers only). Also, the API authentication might be tricky here, as you need to [generate/get the API Access Key on the Enterprise portal](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/ea-portal-rest-apis#generate-or-retrieve-the-api-key) first.

Then, you should [provide that key in the specific format in the request Authorization header](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/enterprise-api#enabling-data-access-to-the-api).

The successful API call will result in a JSON array, provided that you have some Reserved Instance purchases within the target enrollment.

> Depending on your needs, you can switch to the daily grain when querying the API. However, from the practical point of view, it’s rather suitable for an in-depth analysis than for regular monitoring/reporting.

To parse that JSON output into an array we can manipulate, we can use the Parse JSON action, which is [a common data operation in Power Automate](https://learn.microsoft.com/en-us/power-automate/data-operations):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923680/b51f1a33-2ee5-4c7a-b946-43ac5a0bb579.png align="center")

The parsing requires a schema for understanding the structure of a JSON payload. You can generate it by yourself from your sample Reserved Instance usage details payload or use the following schema, which I already prepared:

%[https://gist.github.com/andrewmatveychuk/88ec45ab135e28dadd76fc89d5a95574] 

The resulting array will contain the details for all your Reserved Instance purchases regardless of their utilization levels. Putting all that data into your utilization report might be unnecessary, as we are primarily interested in reservations that are not fully utilized — in other words, reservations with less than 100% utilization. However, as I explain next, it is not always necessary to achieve full utilization to benefit from the savings Azure Reservations provides.

Bad or good, there is no one universal threshold for reservation utilization that can be considered a best practice. That’s because, depending on the reservation terms, the savings from its usage may vary greatly. For example, in one case, the savings from using Azure Reserved VM Instances purchased for one year for a specific Azure VM SKU or instance flexibility size can be approximately 40%. In another case, purchasing reserved instances for a three-year term for the same Azure VM size(s) can result in more than 60% savings if utilized to the full extent.

Technically, if the utilization of your reservation with 60% projected savings drops by 50%, you are still saving 10% using it compared to on-demand costs. In contrast, if the utilization of the 40% reservation drops by the same amount, you are in the red.

Of course, that doesn’t mean you should target such low savings by applying Azure Reservations. For analysis purposes, you can rely on the guidelines provided by Microsoft on the Reservation blade of the Azure Portal:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924279/8adad8fa-2f9e-4375-8e7d-e3ea806e88e9.png align="center")

For the sake of this demo, let’s use the 90% utilization threshold to filter the reservations that require our attention. [The Filter array action](https://learn.microsoft.com/en-us/power-automate/data-operations#use-the-filter-array-action) is to help us here:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925069/ac6470bb-308b-45c6-bd33-1b9fad2782f7.png align="center")

The `avgUtilizationPercentage` property is provided as a float, so be sure to use the corresponding expression to compare it with your value.

Next, the resulting array will still contain data about reservation utilization details for past months that might have little value for us if we already took some action on them. It would be better to focus on the last (or current) month, depending on what day of the month you run this report.

In my case, I run the report on the first day of each month, so it makes sense to see the reservation utilization details for the previous month only:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925652/40e6c029-062a-4db7-9b16-74760d10e6ef.png align="center")

You can achieve that by filtering the `usageDate` property and using the following expression to get last month’s value in the same format as in the API response:

`addToTime(utcNow(), -1, 'month', 'yyyy-MM')`

Optionally, if you want to make your report cleaner, you can remove unnecessary data and select only properties you want to display in the resulting set with [the Select action](https://learn.microsoft.com/en-us/power-automate/data-operations#use-the-select-action):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681926256/f53877c8-5168-47e9-9131-c08c81e61541.png align="center")

Now, when your resulting list of Azure Reservations is ready, you can deliver it to your target audience.

To keep it simple, we will send the flow output as an email notification. For that, we need to convert the resulting list into an HTML table using [the Create HTML table action](https://learn.microsoft.com/en-us/power-automate/data-operations#use-the-create-html-table-action) and then insert the formatted table into the message body:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681926839/2566d1f3-9528-4597-b14b-99888ed5b17b.png align="center")

Lastly, to put the reporting on autopilot, you can set it to run on a schedule using [the Recurrence trigger](https://learn.microsoft.com/en-us/power-automate/run-scheduled-tasks#configure-advanced-options):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681927472/b71ad805-f567-4ae3-9b6b-72558a48d674.png align="center")

Unfortunately, there is no easy way to schedule the trigger for a specific day of a month yet, so you can set it to execute daily with the following trigger condition:

`@equals(utcNow('dd'), '01')`

The final flow structure:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681928036/68200ef0-1c91-4ab5-a34a-12b3dbc9c080.png align="center")

The result of your efforts might look like in the following example:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681928988/357eb18a-1743-4113-b02d-35f53d09a8dc.png align="center")

## In conclusion

Monitoring your Azure Reservations utilization is really important as it explicitly correlates with the numbers in your cloud invoices. It should be a regular part of your [FinOps](https://andrewmatveychuk.com/tag/finops/) practices in Azure, along with analyzing opportunities to purchase new Azure Reservations for the services in use. While Azure Advisor can suggest which reservation to purchase, the ‘rebalancing’ of existing ones is completely up to you. Having some food for thought, like the list of Azure Reservations you don’t use fully, and the list of new reservations to purchase, is the first step in the optimization process.

If you have questions about using Azure Reservations or would like to see more content about Azure [FinOps](https://andrewmatveychuk.com/tag/finops/) practices in general, let me know in the comments! 👇