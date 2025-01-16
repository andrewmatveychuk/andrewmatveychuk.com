---
title: "Maintenance mode for web availability tests in Application Insights"
seoDescription: "Learn how to emulate maintenance mode for web availability tests in Azure Application Insights to avoid false alerts during app updates"
datePublished: Tue Nov 12 2019 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b6j73s000009jrc1ddfxml
slug: maintenance-mode-for-web-availability-tests-in-application-insights
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737018817848/a7087f6a-518d-4fe5-8a92-b517529c26f3.png
tags: azure, devops, powershell, azure-devops

---

## Background

Many teams that host applications in [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/) set up the availability monitoring for their apps using [Application Insights web tests](https://docs.microsoft.com/en-us/azure/azure-monitor/app/monitor-web-app-availability), which can be basic URL ping checks and more advanced multi-step and performance ones. A common approach when creating such tests is configuring alerts to notify or trigger an action on some criteria, i.e., when the tests fail.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922741/9f3bc5b3-262c-4f6f-a65d-c3c0c2145ba1.png align="center")

As a result, people responsible for keeping the application alive are aware when something goes wrong with it and can take remediation actions.

Also, it is common practice to put an application or service in operation into maintenance mode in a monitoring solution before making any changes to it or updating its components so as not to be woken up at 2 a.m. by false-positive alerts on your “pager.”

## Problem

Regarding the Azure App Service applications, in my example, misleading notifications from the web test were triggered during the regular, continuous deployments from Azure DevOps pipelines. This was not good, as each time, the Ops had to verify with the Devs whether the issue was not fake.

Unfortunately, there is no such handy built-in maintenance mode for monitored items in Application Insights as, for example, in [System Center Operations Manager](https://docs.microsoft.com/en-us/system-center/scom/manage-maintenance-mode-overview), where you can suspend all alerting for a specific configuration item or a group of items for some period and not worry about putting them back on monitoring later.

To make it worse, there are even [no PowerShell cmdlets yet to work with the web tests, specifically](https://feedback.azure.com/forums/357324-azure-monitor-application-insights/suggestions/16304431-possibility-to-enable-disable-availability-web-tes) in Az 3.0.0:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923642/ac41f370-a32f-40cf-9949-27c00b9ef6bf.png align="center")

## Solution

Currently, one of the options to interact with ‘Microsoft.Insights/webtests’ resources is to use generic ‘Get-AzResource’ and ‘Set-AzResource’ PowerShell cmdlets. They might not be as convenient as specialized cmdlets, but they still do the job as you can manipulate resource properties.

For example, a script to disable all web tests in a specific resource group might look like the following:

%[https://gist.github.com/andrewmatveychuk/4ddaf20ea130295d7565982d6f5fe6b2] 

To enable them back, you can modify the ‘Enabled’ property to ‘True’ and rerun the script.

Armed with these scripts, you can emulate the maintenance mode for your application in the deployment pipeline:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924199/855acf0b-5383-4970-9f2f-ed67448a367d.png align="center")

You can use the Azure PowerShell tasks to disable the tests before deployment and enable them back afterward.

Also, don’t forget to configure the run conditions for the enabling task so you have your monitoring turned on even if a previous task, e.g., a deployment, failed:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924790/1a65e252-eebf-4e57-b0bb-cccb195bdfa0.png align="center")

A sample YAML snippet for this:

%[https://gist.github.com/andrewmatveychuk/2ffe3bd1a7d168cdb611128f3bfbb6fc] 

Also, I didn’t want to use [ARM REST APIs](https://docs.microsoft.com/en-us/rest/api/azure/) in my solution as their format usually makes the script logic hard to understand, and IMHO, it is a complete nightmare for future solution maintenance.

How do you monitor your apps in production and put them in maintenance mode? Share your experience in the comments!