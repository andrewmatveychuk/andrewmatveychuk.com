---
title: "How to send custom Azure Automation Runbook logs to Log Analytics"
seoDescription: "Send Azure Automation Runbook logs to Log Analytics for efficient storage, retention, and advanced querying"
datePublished: Mon Feb 26 2024 13:00:03 GMT+0000 (Coordinated Universal Time)
cuid: cm5dnn3jt00030amo0him591i
slug: how-to-send-custom-azure-automation-runbook-logs-to-log-analytics
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736868359551/640f4294-fd01-4252-86e4-ea2d5b56c699.png
tags: azure, logging, powershell, azure-monitor

---

In this blog post, I will explain why it might be helpful to implement custom logging in your Azure Automation runbooks, why to use Log Analytics as a log destination, and how to implement a logging framework that can be scaled and shared by your runbooks.

> Note. I will discuss PowerShell runbooks here, but many concepts described below also apply to other Azure Automation runbook types.

## Why use custom logging for Azure Automation runbooks?

Azure Automation accounts support extensive capabilities for tracing and logging runbook execution results. If you are familiar with [PowerShell streams and how they work](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_redirection), you can also use them in your runbooks. All regular PowerShell logging practices with verbose, debugging, warning, and error outputs can be used in Automation runbooks without any modifications. The only thing to remember is that you need to explicitly enable their logging in the runbook job history for some streams.

However, the runbook job history is available by default only for [the last 30 days](https://learn.microsoft.com/en-us/azure/automation/automation-managing-data#data-retention). So, if you want to keep your logs longer or run some queries on them, you can send them for long-term storage to a Storage account, a Log Analytics workplace, or an external system [using diagnostic settings](https://learn.microsoft.com/en-us/azure/automation/automation-manage-send-joblogs-log-analytics). Just remember to include the JobStreams log category in your settings.

Those system options for runbook logging will be enough for most use cases. However, those default logging options can add too much irrelevant data to the logs. Plus, [extracting application-specific details from those system logs](https://learn.microsoft.com/en-us/azure/automation/automation-manage-send-joblogs-log-analytics#sample-queries-for-job-logs-and-job-streams) might pose some challenges, as there are no technical restrictions on what application-specific information should be present in those logs and how it should be formatted. So, can we implement *structured* custom logs, and where should we store them?

## Why send your custom runbook logs to Log Analytics?

Although there are no technical limitations on where to store your custom logs in Azure, some Azure services are better suited for that purpose than others. For instance, the available storage options in Azure resource diagnostic settings list Log Analytics workspaces and Storage accounts as the two most common places to store log data.

> You can certainly use any form of storage or database for your logs, as you might have specific requirements for your solution. However, if you don’t need to use Azure SQL Database, Azure Cosmos DB, or any other functional storage solution specifically, I suggest you start with one of the default options present in Diagnostic settings.

Log Analytics workspaces store data in a tabular format and provide extensive query capabilities. So, if you need to build some analytics on top of your logs or use them extensively for troubleshooting, definitely consider them first.

Storage accounts can store data in different forms, such as files, blobs, and tables, but they lack any reporting and analytical capabilities at the service level. Using data stored in them requires using some external solutions, which can make your overall design more complex. They are a suggested log storage destination in scenarios when you need to keep a log archive that is rarely accessible and you want to make it as cheap as possible.

## How to send data from PowerShell to a Log Analytics workspace?

The current suggested approach for sending data to a Log Analytics workspace is to use [the Logs Ingestion API in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/logs-ingestion-api-overview). Compared to the deprecated Data Collector API, now you can create a Data Collection Endpoint (DCE) resource in your Azure subscription, which serves as an entry point for sending your data to a workspace. Plus, as a part of the required setup, you create Data Collection Rules (DCR) that group transformation logic you can apply to the incoming data before it is sent to the destination workspace.

Unfortunately, Microsoft hasn’t yet provided a PowerShell [SDK to abstract REST calls to the Log Ingestion API](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/logs-ingestion-api-overview#client-libraries). However, the provided sample PowerShell code can be easily adapted for making calls to the API from a runbook like the following:

%[https://gist.github.com/andrewmatveychuk/3a747949610efb23fd39617395027e68] 

As you can see from the runbook, it authenticates to Azure [using a system-assigned identity for the Automation account](https://learn.microsoft.com/en-us/azure/automation/enable-managed-identity-for-automation). That identity should be assigned [the Monitoring Metrics Publisher role](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#monitoring-metrics-publisher) so the target DCR can push data to it. That way, you don’t have to maintain a separate application registration in your tenant to access your collection rules.

> For sure, you can authenticate to your DCR rules using custom application registrations, as presented in official samples, if you want to make your access model as granular as possible. However, it might be less practical than using managed identities, as you must maintain those registrations and rotate their access credentials.

After the authentication, it uses the obtained Azure context to acquire an access token for a specific API, which is Azure Monitor in our case. The token is immediately converted to a secure string [to be used by the follow-up Invoke-RestMethod cmdlet](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod?#-token). Next, it makes the preconfigured API call and outputs response results.

Provided that you constructed the correct JSON payload that your DCR expects from your PowerShell object, you will see new log entries in your custom workspace table in a moment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922713/20bbc883-8a12-4a3f-a51e-551af1e5ac9f.png align="left")

> Please note that the initial population of the destination custom table might take a while. After it is set up and indexed, all further entries can be queried much faster.

You can stop here or make your runbook logging solution more robust and reusable. For example, you can extract the logic for pushing your log entries into a separate function packed in a custom PowerShell module that you can import into your Automation accounts. That way, you can make your runbook code cleaner and reuse your logging function as a regular PowerShell cmdlet in your other runbooks.

%[https://gist.github.com/andrewmatveychuk/3c79a5a94a8254b43a185b05cd93d5a9] 

In addition to that, you can also look into defining [custom PowerShell classes](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes) to improve the type safety of your log entry object and make your logs more structured, as with [PSCustomObject](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pscustomobject), you can assign its properties any values, which may lead to running into errors on the DCR transformation step due to mismatch in expected input format:

%[https://gist.github.com/andrewmatveychuk/f66e81864f2a96c8ff3a772b38cdc009] 

As you can see from that sample code, you can define [custom PowerShell enum types](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_enum) to limit the number of allowed values for specific log entry properties used for categorization. Also, explicitly defining class property types will help you correct formatting errors when creating log entry objects in your runbooks, not when a receiving DCR transformation stream rejects those entries.

Now, your updated runbook that sends custom logs to a Log Analytics workspace might look like this:

%[https://gist.github.com/andrewmatveychuk/29064cb51f7c126698bbd121ecc0d404] 

> There are also some community solutions for that, like the [AzLogDcrIngestPS module](https://github.com/KnudsenMorten/AzLogDcrIngestPS) by Morten Knudsen, which I honestly recommend checking, that allows you to automate all the work around configuring the resources required for the Log Ingestion API. However, introducing an unofficial module dependency only for a small portion of its functionality, like only sending logs, might not always be desirable.

Have you implemented custom logging in your Azure Automation runbooks? Did you find it helpful, and why? Share your opinions in the comments below 👇