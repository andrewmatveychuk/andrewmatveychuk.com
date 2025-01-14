---
title: "How to use change history in Azure Monitor Workbooks"
seoDescription: "Use Azure Monitor Workbooks with Azure Resource Graph to track and analyze configuration changes for better troubleshooting and incident management"
datePublished: Mon Aug 07 2023 11:00:27 GMT+0000 (Coordinated Universal Time)
cuid: cm5b55owx000809mja04xeey5
slug: how-to-use-change-history-in-azure-monitor-workbooks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736867563882/ac1c9114-6c53-4549-b965-40b18395b3f2.png
tags: howtos, azure-monitor, it-operations

---

Anyone who has worked in IT Operations long enough knows that many incidents and service outages occur due to recent application configuration changes. If an application has an SLA, incident handling routines often explicitly instruct users to check for such changes as one of the first troubleshooting steps.

Despite Azure providing many options for tracking and analyzing resource configuration changes, those tools are [spread across different services and sections of the portal](https://learn.microsoft.com/en-us/azure/azure-monitor/change/change-analysis#change-analysis-architecture). While some of them can be used out-of-the-box, others, like [Change analysis in VM insights](https://learn.microsoft.com/en-us/azure/azure-monitor/vm/vminsights-change-analysis), [Change Tracking in Azure Automation](https://learn.microsoft.com/en-us/azure/automation/change-tracking/overview), or saving Activity logs to Log Analytics workspaces, require additional configuration before use.

Last year, Microsoft announced [the availability of multiple Change Analysis features](https://techcommunity.microsoft.com/t5/azure-observability-blog/change-analysis-ga-announcement/ba-p/3613302). What makes them great is that they use [Azure Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/how-to/get-resource-changes) under the hood, which has an extensive list of SDKs. From the practical perspective, that allows you to pull the data about changes from the graph into many Azure-native monitoring solutions and third-party ones. Here, I will explore how you can view the history of such configuration changes in Azure Monitor Workbooks.

## Pull data about changes in Azure Monitor Workbooks

In Azure Monitor Workbooks, you can use [the Azure Resource Graph data source](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#azure-resource-graph) to execute supported KQL queries and display their results in various formats. For example, you can use the following query to pull all changes for the [last 14 days](https://learn.microsoft.com/en-us/azure/governance/resource-graph/how-to/get-resource-changes?#best-practices) scoped to a resource group:

%[https://gist.github.com/andrewmatveychuk/9ab0b3d8a05e63e81600fe60c77dcc43] 

The resulting table might look like the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922771/c03b081b-52ce-4a96-a996-6a77415005b2.png align="center")

Unfortunately, the [Change Analysis data source](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#change-analysis-preview), which is in preview as of now, scopes its results to a single resource only. That might not be so helpful in cases when a change in another resource causes an alert linked to your target resource. For example, an error in your web app might be caused by a changed or deleted certificate in a Key Vault.

## List active alerts in Azure Monitor Workbooks

Obviously, having a list of active alerts in the same workbook alongside the configuration changes would be helpful to correlate between them. Previously, there was a separate data source for pulling the information about Azure Monitor alerts, but now [the alert info is available via Azure Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/samples/samples-by-category?tabs=azure-cli#view-recent-azure-monitor-alerts). The following query will return the list of all active Azure Monitor alerts in the target resource group:s

%[https://gist.github.com/andrewmatveychuk/dbe46caabe9b89df1cc6c338600b927b] 

A sample result of that query:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923856/cb1a7e01-8fa4-47f9-9c81-3fc0b9c2d488.png align="center")

Another option to get the alert information is to [pull it with an Azure Resource Manager query](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-commonly-used-components#use-azure-resource-manager-to-retrieve-alerts-in-a-subscription).

## Improve troubleshooting experience with Azure Monitor Workbooks

Sometimes, you might need to get back to resolved alerts to review and analyze them. You can achieve that by using parameters in Azure Monitor Workbooks.

For example, you can [add parameters to display alerts that fired during the last N hours/days](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-time) and [optionally include resolved alerts to your results](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-dropdowns). After that, you can reference those parameters in your query to make it more dynamic:

%[https://gist.github.com/andrewmatveychuk/6528ac35aa295ff215a132a09a81b5f8] 

Also, you can improve your user experience by [formatting the results with colorful icons](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-commonly-used-components#traffic-light-icons) and using [column formatting](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-grid-visualizations#style-a-grid-column):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924543/655274b3-9e16-4309-8c14-f54b9515d9a9.png align="center")

> If you need to analyze changes preceding alerts fired beyond the last 14 days, you will need to [schedule exporting those logs to some external store like Log Analytics](https://learn.microsoft.com/en-us/azure/governance/resource-graph/tutorials/logic-app-calling-arg) and modify your workbook to pull the change info from a workspace instead of Azure Resource Graph directly.

Because, from the troubleshooting perspective, it makes sense to look into the changes that happened before the alert was fired and not after, you can take an extra step and make the results in your change history view depending on a specific alert you select in your alerts view. For that, you can [define a workbook parameter that is populated when you click on a row in your alert grid view](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-configurations#set-up-a-grid-row-click) and use it in a follow-up change history query to filter the changes that happened earlier than the alert fired:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925134/34704399-3e9f-421e-8989-5366c547441f.png align="center")

%[https://gist.github.com/andrewmatveychuk/d7722c8bdfc4a2ad5d82ad04e178adda] 

The improved change history view:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925796/5d9f1e55-2002-4974-91e8-1f5b7a7c3ec3.png align="center")

You can also extend your workbook with information about [resource](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#azure-resource-health) and [workload](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#workload-health) health in the target scope to immediately check if an outage is related to resource availability.

Feel free to check [my GitHub repository](https://github.com/andrewmatveychuk/azure.monitor/tree/master/workbooks) for the complete sample workbook template for correlating between alerts and configuration changes in Azure.

What is your experience analyzing changes that caused application performance or availability issues? What tools do you use the most to review configuration changes in Azure resources? Please share your thoughts in the comments 👇