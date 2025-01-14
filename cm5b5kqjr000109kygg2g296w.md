---
title: "How to audit Azure Hybrid Benefit usage with Azure Workbooks"
seoTitle: "Audit Azure Hybrid Benefit with Azure Workbooks"
seoDescription: "Use Azure Workbooks to audit Azure Hybrid Benefit, monitor license consumption, and optimize resource allocation"
datePublished: Thu Jan 05 2023 15:30:19 GMT+0000 (Coordinated Universal Time)
cuid: cm5b5kqjr000109kygg2g296w
slug: how-to-audit-azure-hybrid-benefit-usage-with-azure-workbooks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736870974612/20bc419f-1660-4b89-b44d-050b4c1f313b.png
tags: azure, finops, azure-monitor

---

> **UPDATED**. It seems that [Microsoft borrowed my idea of analyzing Azure Hybrid Benefit usage with Azure Monitor Workbooks](https://azure.microsoft.com/en-us/updates/public-preview-assess-cost-optimization-opportunities-using-new-workbook-template-in-azure-advisor/) and included the corresponding charts in [the Cost Optimization workbook](https://learn.microsoft.com/en-us/azure/advisor/advisor-cost-optimization-workbook#azure-hybrid-benefit) in Azure Advisor. Feel free to check it for AHUB numbers and recommendations in your environment. It’s great to see that they listen to customer feedback!

In one of my previous blog posts, I wrote about [enabling Azure Hybrid Benefit at scale using Azure Policy](https://andrewmatveychuk.com/audit-and-enable-azure-hybrid-benefit-using-azure-policy). Today, we will explore how you can keep track of actual license counts consumed by Azure services with that license benefit turned on. Moreover, we will try to analyze those resources for optimal benefit usage.

> If you want to skip the details and just get the Workbook for auditing Azure Hybrid Benefit usage, check for its template in [my GitHub repository](https://github.com/andrewmatveychuk/azure.monitor/tree/master/workbooks).

## What are Azure Workbooks?

[Azure Workbooks](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview) (also known as Azure Monitor Workbooks) is an excellent tool for visualizing data in the Azure Portal and sharing prebuilt reports with other users. They allow you to pull data from different data sources, extract some meaningful insights from that data, and present them to users in various formats. If you are familiar with [Azure Dashboards](https://learn.microsoft.com/en-us/azure/azure-portal/azure-portal-dashboards), you can think of Azure Workbooks as a more advanced reporting solution.

There is enough documentation to [start working with Azure Workbooks](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-getting-started), so I will focus more on the practical case rather than explaining Workbooks’ functionality. If you are not familiar with them, in addition to the official documentation, I recommend checking the following videos produced by the people who created that great Azure service:

* [How to use Azure Monitor workbooks](https://www.youtube.com/watch?v=Z5xRyy3HB8U)
    
* [How to build Azure Workbooks using logs and parameters](https://www.youtube.com/watch?v=EC7n1Oo6D-o)
    
* [How to build a graph and use links in Azure Workbooks](https://www.youtube.com/watch?v=ODHczGpGmEQ)
    
* [How to build tabs and alerts in Azure workbooks](https://www.youtube.com/watch?v=3XY3lYgrRvA)
    

Working with most supported Azure Workbook data sources requires using [Kusto Query Language (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/) to query data and JSON to parse API responses, so some knowledge of those technologies will also be desirable.

> On a personal side, I must admit that I overlooked Azure Workbooks functionality for a long time because I could usually gather required data using PowerShell much faster. However, doing everything with a familiar tool only won’t help you explore other options, so you understand their pros and cons and can use them effectively. Therefore, I just decided to use this practical case as a learning opportunity for myself.

## What data are we going to use, and how can we bring it to a Workbook?

For the sake of simplicity, let’s focus on [Azure Hybrid Benefit for Windows Server VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing) only. Technically, we need to get the list of all Azure VMs with Windows Server OS and Hybrid Benefit enabled. For that task, we can leverage the power of [Azure Resource Graph](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#azure-resource-graph). A sample Graph query might look like this:

%[https://gist.github.com/andrewmatveychuk/019106dd6c6a1f5232dd8b2c939750eb] 

That query looks up for all Azure resources where the resource type is Azure VM, the OS image publisher is Microsoft Windows Server and the corresponding VM property controlling the license benefit usage is equal to ‘[Windows\_Server](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing#template)’.

> Please note that [KQL is case-sensitive](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/#what-is-a-query-statement), and I use [case-insensitive operators](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/equals-operator) as value naming is not always consistent across all Graph tables.

As Azure Hybrid Benefit for Windows Server VMs entitles you to cover the software costs based on [the number of core licenses](https://learn.microsoft.com/en-us/windows-server/get-started/azure-hybrid-benefit#getting-azure-hybrid-benefit-for-windows-vms-in-azure) you own, we will summarize query findings as the counts of discovered VM sizes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922554/15cb47bb-554c-4ef3-a91b-6daa3961d90d.png align="center")

Unfortunately, Azure VM size names don’t always correctly reflect the number of vCPUs according to [the Azure VM sizes naming convention](https://learn.microsoft.com/en-us/azure/virtual-machines/vm-naming-conventions). So, just parsing the VM size names for CPU counts won’t work. A suggested approach to get such data is to use [the Resource SKUs API](https://learn.microsoft.com/en-us/rest/api/compute/resource-skus/list). As that API is a part of Azure Resource Manager REST APIs, we can use an [Azure Resource Manager query](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#azure-resource-manager) in our workbook to pull the data from it:

`/subscriptions/{Subscription:id}/providers/Microsoft.Compute/skus?api-version=2021-07-01&$filter=location eq '{Location}'`

Note that the API is subscription-scoped, and we provide the subscription ID via a [Workbook parameter](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-parameters). We also scope the data to a particular Azure region, which is also a parameter, to limit the amount of data the query returns and avoid duplicates.

> Theoretically, if your Azure VMs are deployed across many different regions, you might miss data about VM sizes unavailable in the region scoped by the query. However, in practice, the VMs are usually deployed to a limited number of customer-selected Azure regions that support the same set of required VM SKUs. If that is not the case, you might need to pull the list of supported VM sizes for all regions and remove duplicate data from the results.

As the mentioned API returns [a JSON payload containing SKU data for different resource types](https://learn.microsoft.com/en-us/rest/api/compute/resource-skus/list?tabs=HTTP#lists-all-available-resource-skus-for-the-specified-region), we can transform the result with [JSONPath](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-jsonpath) expressions:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923394/dab3c6a4-b23c-43bc-b291-a35f11bc906a.png align="center")

Firstly, we filter items using Azure VM as the resource type. Secondly, we select only the VM size properties we want: a VM SKU name and the number of vCPUs. A sample query output might look like the following table:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924039/b8eee31d-6bdd-494e-a3e2-53ff663289a3.png align="center")

## Can we combine data from different data sources?

Now, we have two pieces of data – the counts of used Azure VM sizes with Azure Hybrid Benefit enabled and the number of vCPUs for each VM size. It would be great to combine that data by [joining tables in SQL](https://www.w3schools.com/sql/sql_join.asp). Fortunately, we can do that using [Azure Workbook Merge](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#merge) functionality. It allows you to perform different types of joins, so here we will use the left outer join to combine all items from our Azure VMs grouped by size (right table) with the matching items from the Azure VM SKUs list (left table):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924686/52d8b94d-6173-456b-8804-6ef5ed6c702a.png align="center")

Also, we can select the columns we want to see in the resulting data set and remove duplicate ones. Additionally, the Merge data source allows you to extend your data set using calculated columns. For example, you can use expressions to calculate the total number of used vCPUs per VM size:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925293/b2a360f6-3d53-4f0f-9a6f-2b9cb559e5a3.png align="center")

The resulting table might look like the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925918/6363ecd0-941c-49ec-a1c4-37b5cbef0b52.png align="center")

However, those results are intermediate only, as [the number of CPU cores doesn’t directly translate to the number of core licenses you need](https://learn.microsoft.com/en-us/windows-server/get-started/azure-hybrid-benefit#licensing-prerequisites). So, how do we know how many core licenses are consumed by each VM size?

## Do you really benefit from Azure Hybrid Benefit?

As it turns out, regardless of the number of vCPUs, each [Windows Server Azure VM with less than 8 vCPUs always consumes 8 core licenses](https://learn.microsoft.com/en-us/windows-server/get-started/azure-hybrid-benefit#number-of-licenses). Servers with more than 8 vCPUs can stack more licenses on top of that minimum, but the license count should be rounded up to the nearest bigger integer divisible by 8. So, a 12-CPU VM will still consume 16 core licenses and so on.

To determine the actual number of required core licenses for our Windows Server VM fleet, we can use an additional calculated column with the following conditional logic:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681926503/776c157f-a485-4e5c-a242-05f1f5e48b85.png align="center")

Those expressions round up the number of licenses to:

* 8 if an Azure VM has less than 8 vCPUs,
    
* the nearest bigger integer divisible by 8 in all other cases.
    

So, now we can see precisely how many core licenses are needed for each Azure VM size group without doing any math manually.

On top of that, we can do another trick with calculated columns and [add icons](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-grid-visualizations#icons-to-represent-status) signaling if a specific VM size is using Azure Hybrid Benefit efficiently from the license point of view:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681927099/974c53ef-ea0a-47b1-92f4-38cde53ee817.png align="center")

The condition here compares the licenses required to the number of vCPUs and produces a successful output if those numbers are equal. All other cases are considered suboptimal.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681927765/0c4de4ba-129e-496e-bd81-f8be34b8acd6.png align="center")

That visualization can help us quickly understand which VM sizes in use don’t fully benefit from the license benefit. This can be especially helpful if you don’t have enough core licenses to cover all your Azure VMs, and it makes sense to redistribute them for maximum savings — enable Azure Hybrid Benefit only for VMs with 8 vCPUs and more.

## Can we make our solution more user-friendly and reusable?

At this point, our resulting table contains all the details we might need to audit Azure Hybrid Benefit usage by Windows Server VMs in a subscription. However, what if we want top-level indicators based on that data to provide quick insights without digging into the records? For example, let’s add the total number of consumed core licenses and the total number of vCPUs to see how effectively we use licenses in general.

Instead of running additional queries in our workbook to fetch that data, we can [reuse the existing dataset using the Merge data source](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-commonly-used-components#reuse-query-data-in-different-visualizations). It allows you to make a copy of another query results so that you can present them in a different format:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681928362/042df27e-a9a0-442f-86d0-dd31e30c2c85.png align="center")

Then, we can remove unnecessary columns, choose a different visualization option, and select the category to focus on:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681929313/508ef739-c113-4450-b71b-88d223309599.png align="center")

As the sample figures above show, using licenses in a lab environment is far from efficient.

Apart from those improvements, we can easily enable data export in Azure Workbooks, so other non-technical people can export the detailed results if they want to do some analysis with external tools:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681929989/058298d0-2b0c-423f-92fc-5eac602e23e2.png align="center")

Azure Workbooks can be exported as [Workbook templates or ARM templates](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-automate). Those templates can be imported into other environments(subscriptions) without any substantial preparation. This allows you to share them with other people as ready-to-consume solutions that don’t require technical expertise to use, making them a perfect case for solution adoption.

## What is next?

Finally, you can grab the ready-to-use Workbook for auditing Azure Hybrid Benefit usage from [my GitHub repository](https://github.com/andrewmatveychuk/azure.monitor/tree/master/workbooks), import it into your Azure subscription, and check the license numbers in your environment. Currently, it covers the cases for Azure Hybrid Benefit for [Windows Server VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing), [SQL Server VMs](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change), and Windows Client VMs (aka [Multitenant Hosting Rights](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/windows-desktop-multitenant-hosting-deployment#subscription-licenses-that-qualify-for-multitenant-hosting-rights)). Feel free to adjust it to your needs and add missing functionality for other Azure Hybrid Benefit use cases.

Got a question? Post it in the comments below and let me know if that guide was helpful for you 👇

**Update.** The workbook was also updated to provide information about the license benefit usage by [Azure SQL Database and SQL Managed Instance](https://learn.microsoft.com/en-us/azure/azure-sql/azure-hybrid-benefit).