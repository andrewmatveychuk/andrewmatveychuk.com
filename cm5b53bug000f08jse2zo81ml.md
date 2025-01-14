---
title: "Azure Savings Plans vs. Azure Reservations"
seoDescription: "Compare Azure Savings Plans and Reservations to determine the best cost optimization for your workloads"
datePublished: Mon Jul 17 2023 11:00:24 GMT+0000 (Coordinated Universal Time)
cuid: cm5b53bug000f08jse2zo81ml
slug: azure-savings-plans-vs-azure-reservations
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736870919330/2b79e3c3-e340-4d1d-8482-510304ab5b27.png
tags: azure, finops

---

Last fall, Microsoft announced a new cost-saving option for Azure workloads called [Savings Plans](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/). Many customers who already were using [Reservations](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/) (aka Azure Reserved Instances) to save on their cloud costs mistakenly perceived Savings Plans as a replacement for them. So, let’s try to demystify Savings Plans and whether you should migrate to them from Reservations.

## Key differences between Azure Reservations and Savings Plans

To make informed decisions when [choosing among different cost-saving options in Azure](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/decide-between-savings-plan-reservation), we first need to understand their differences, summarized in the following table:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922662/145fcd64-7a6c-4c1c-b680-a1cbc79f6b3b.png align="center")

The first difference to pay attention to is how those two options are applied. While a reservation is bound to one specific Azure region you chose when purchasing it, the saving plan will automatically target eligible resources in all regions in the assigned scope, which can be a shared scope, a management group, a subscription, or a resource group for both saving options. For example, your resource group contains two virtual machines deployed in two different regions. With Azure Reservations, you must purchase two of them bound to the respective regions and scope them to your resource group. Suppose you migrate any of those VMs to a region different from the original one. In that case, the corresponding reservation won’t be applied anymore, and you will be billed on a pay-as-you-go basis for them, plus still paying for the unused reservation(s). In the case of Savings Plans, the lower savings plan prices will still apply to those VMs regardless of the region.

Apart from that, while [Reservations apply only to specific resource SKUs](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/reservation-discount-application) you define during their purchase, Savings Plans will apply their ‘discount’ to all resource sizes (with some exceptions). For instance, if you resize your VM, you will likely lose the benefits from the reservation unless the new VM size is from the same SKU family and the reservation was configured for [instance size flexibility](https://learn.microsoft.com/en-us/azure/virtual-machines/reserved-vm-instance-size-flexibility).

Secondly, [Azure Savings Plans target only a limited list of compute resources](https://azure.microsoft.com/en-us/pricing/offers/savings-plan-compute/#benefits-and-features), which are Azure Virtual Machines, Azure Container Instances, Azure Functions Premium Plans, Azure App Service (Premium v3 and Isolated v2 only), and Azure Dedicated Host at the time of writing this. That list might be extended in the future, so please refer to the official docs for up-to-date information. In contrast, Azure Reservations, in addition to compute resources, can also [target storage and a wide range of PaaS resources](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/save-compute-costs-reservations#charges-covered-by-reservation) such as databases, various data and analytics solutions, etc.

Thirdly, while [Reservations are partially refundable and exchangeable](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations), [Savings Plans are not](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/cancel-savings-plan). That difference will disappear in the future, as Reservations purchased starting from Jan 1, 2024, will practically no longer allow exchanges.

Lastly, the level of potential savings. Please don’t be deceived by those numbers, as they are the maximum you can achieve with specific conditions like a certain VM size, region, commitment duration, etc. The actual numbers might vary significantly, so it’s always good to do your homework and understand your current savings and the potential ones *in your specific case*.

I intentionally omitted some similarities between those two options in my comparison to focus on the most critical factors that can affect your decision about going with Savings Plans, Reservations, or both. If you need full details, I encourage you to check the official documentation for them, as their usage policies might be updated, and new services might be added to the list of applicable savings:

* [Azure Reserved Instances](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/)
    
* [Azure Savings Plans](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/)
    

Now, let’s look at how to use those options in practice.

## Practical application of Savings Plans

Despite Microsoft doing a great job presenting that new saving option to its customers, I still observe some confusion when people start talking about using Savings Plans in practice. Most of it originates from the misunderstanding that Savings Plans intend to replace Reservations. In fact, Savings Plans should be perceived as a complementary cost optimization tool. Choosing the proper tool depends on your Azure environment and workload run patterns. Let’s review a few workload examples and see what the most appropriate cost-saving solution we can use for them. I will use compute workloads to compare apples to apples, as Savings Plans currently cover only them.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923838/03aa6f54-eba4-4fd9-ab46-3585d2753298.png align="center")

In the first case, your environment consists mostly of IaaS components deployed in a few Azure regions. Your applications run on VMs that are mostly persistent resources with stable capacity demand and a relatively long lifetime (years). The movement of VMs between regions and their vertical scaling is rare, which is performed manually when a need arises. That is a typical picture of long-running, persistent infrastructure with no sudden changes in resource utilization. Azure Reservations would be the first consideration, as they can help you achieve greater savings when applied to such stable workloads.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924479/38eee0f2-dbfb-4d05-8cf5-bd78354dc1db.png align="center")

In the second scenario, you are in the migration phase when your organization moves its application from VMs to more scalable PaaS and CaaS solutions. You still have many VMs in your hands, but at the same time, their list and configuration constantly change as you migrate to App Services, Functions, containers, etc. The landscape of those new resources is also changing and evolving as product teams learn, adapt and adjust to new business needs. Azure Savings Plans would be a preferable saving option here, as they apply to all those compute resources regardless of their type, size and location. The common denominator is the hourly commitment to consume compute resources for X dollars. Whether that money was spent on running a VM or App Service doesn’t matter. In any case, the usage less or equal to X will be billed at lower prices.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925209/103f1c50-9976-46d1-bb9c-faa787006846.png align="center")

A combined scenario is more likely to be observed in large environments. You have different parts of your infrastructure at different levels of cloud adoption. Some pieces are old apps with specific technical and regulatory compliance requirements that run on VMs and shouldn’t be touched. Others are all bells and whistles of the most recent cloud services – autoscaling, serverless, microservices, etc. Plus, there are plenty of areas between those two edge cases. Here you are likely to weigh your cost-saving options and stack them to have the optimal solution. Reservations can cover stable compute resources. The dynamic ones with steady spending can benefit from Savings Plans. If the scope of a reservation and savings plan overlaps, the reservation discount will be applied first, and the savings plan reduced pricing second.

For example, achieving 100% utilization for Azure Reservations in a dynamic environment can be quite challenging – you need to [monitor their utilization](https://andrewmatveychuk.com/how-to-monitor-azure-reservations-utilization), analyze their resource coverage, rebalance existing underutilized reservations, and perform those activities regularly. Besides, it’s most likely that your savings from using Reservations at scale will differ from the ‘promised’ 72% maximum. If you analyze potential savings from using Savings Plans, you might find out that they can provide you with relatively the same level of savings in your environment, but you will need to spend far less time on administering them. Or, you can reduce the list of resources covered by Reservations to decrease the management overhead and use Savings Plans to cover the emerging difference.

Of course, those examples might look simplistic, and the real-life cost optimization strategy will likely account for more requirements. The important point is to understand your environment and always run the numbers before siding with specific cost optimization approaches. Just looking for higher potential savings without understanding the context may lead to an increase in FinOps management overhead, diminishing your returns.

Have you started using Azure Savings Plans in your environment? Were they beneficial, and how? Share your experience in the comments 👇