---
title: "How to ensure proper configuration for your Azure resources"
datePublished: Thu Jan 16 2020 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5o2pv7e000b09mq558hbhzs
slug: how-to-ensure-proper-configuration-for-your-azure-resources
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922676/c485afe0-9025-48b9-b6e5-b1079a9bc777.png
tags: azure, backup, azure-sql-database, azure-policy

---

A common task for Azure administrators is to ensure that resources in their Azure subscriptions are configured according to some business requirements or standards’ regulations. To help administrators with this kind of task, Microsoft implemented many [built-in Azure Policies](https://github.com/Azure/azure-policy/) and policy sets, aka Policy Initiatives, that can audit Azure resources for security and redundancy best practices and regulatory compliance and even look into the internal configuration of your VMs with Azure Policy’s Guest Configuration.

All these built-in policies are a great source of inspiration for what you can achieve with this Azure service and how it can make your job much easier. They can be as simple as enforcing mandatory tags or specific [resource naming patterns](https://andrewmatveychuk.com/how-to-enforce-naming-convention-for-azure-resources), or more complex, which [modify](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#modify) the configuration of resources or [deploy](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#deployifnotexists) missing parts. Today, however, let’s focus our attention only on the auditing part.

## Azure Policy effect – ‘AuditIfNotExists’

The most used ‘[Audit](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#audit)’ policy effect is pretty simple to understand as it only creates a warning event if a resource is not compliant. It has no additional parameters or configuration options and is limited to the policy rule’s condition. The ‘[AuditIfNotExists](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#auditifnotexists)’ condition is more interesting as it can verify the condition of related components for the resources that match the ‘if’ policy rule part. So, as in [the official example](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#auditifnotexists-example), you can check whether the virtual machine extension resource exists alongside the VM resource and whether the extension has specific attributes that identify it as an Antimalware one.

So, let’s take action and consider a practical task when you need to ensure that your Azure SQL databases have proper retention policy configurations to comply with an internal backup policy.

## Example of auditing the configuration of Azure SQL database retention policies

Azure SQL databases are a convenient relational database-as-a-service solution that allows you to focus more on application development and put less effort into maintenance. However, they still require suitable operational tasks, such as regular backups.

Although Microsoft implemented a cool feature called [automated backups for Azure single/pooled databases](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-automated-backups?tabs=single-database), enabled by default for business-critical systems, it is common to require backups for much [longer periods](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-long-term-retention). So, you should usually have short-term and long-term retention policies configured according to your backup policy requirements for production databases.

Also, from an operational perspective, the databases you manage might have different criticality for your business continuity. Some can be highly critical, meaning you can go out of business if you lose them. In contrast, others will have no or little value and can go offline without any significant impact on core business operations.

Apart from that, in IT Service Operations, it is a widespread practice to define those criticality levels and assign all services in operation to one of the levels. This way, you can define operational service requirements, such as SLO, RTO, RPO, etc., in connection to the service criticality levels and not for each application individually so that you have fewer rules to run them. Putting new services in operation also becomes easier as you don’t have to develop a new set of rules each time and can agree on service criticality with business people.

Imagine having three service criticality levels: high, medium and low. You must ensure that the Azure SQL databases related to your services have proper retention policy settings to ensure compliance with the backup policy for those different criticalities.

So, first of all, you should have some information about the criticality level assigned to your databases. The most apparent approach to achieve that is to use Azure metadata tags and assign each database a specific tag with a defined criticality level value. There are plenty of ways to do that: create that tag during deployment from the ARM template, create an [Azure Policy to deny the deployment of resources without a specific tag](https://docs.microsoft.com/en-us/azure/governance/policy/samples/enforce-tag-value) or [inherit the corresponding tag with its value from the resource group level](https://andrewmatveychuk.com/automatic-tagging-for-azure-resources). The latter approach might be especially preferable if you use resource groups as logical boundaries for your applications or services and define the criticality of the whole service and not its parts, as I mentioned previously.

Next, when all your production databases are marked with criticality level tags, you can create custom Azure policies to audit their backup settings.

For example, you can use the following Azure Policy rule snippet for auditing the configuration of the Azure SQL database short-term retention policy:

%[https://gist.github.com/andrewmatveychuk/2ac635f06b12d3ecba251b5454653e70] 

Or the following snippet for the long-term retention policy:

%[https://gist.github.com/andrewmatveychuk/d24dc76d7633b2c36d1a97865c88546c] 

Both examples contain the ‘if’ rule condition check for an Azure SQL database, which has a specific criticality value, and the ‘details’ section of the ‘AuditIfNotExists’ condition, which evaluates the existence of particular retention policy resources.

As you might notice, I provided tag value and retention settings as policy parameters to make the same policy definitions reusable for different criticality levels. Also, you should consider the logical evaluations in the ‘AuditIfNotExists’ condition: the rule will create warning events for all resources that don’t satisfy the ‘existenceCondition’.

To make such policies more convenient, you can group them into policy initiatives and assign them to target scopes. For example:

%[https://gist.github.com/andrewmatveychuk/d2a5b7b6ec2175c2ad15574b724aaf38] 

When designing your policy initiatives, note that their parameters should be provided as an object type for your solution to work. Besides, if you define your custom Azure policies and initiatives in ARM templates, as I do, remember to compose their parameter files to pass objects.

Check out [my repository for Azure Policies on GitHub](https://github.com/andrewmatveychuk/azure.policy) for complete code samples.

## What is more

Despite the relative simplicity, the mechanics of the ‘AuditIfNotExists’ condition can be compelling and utilized for almost any configuration scenario. Basically, you can verify the configuration of nearly all Azure resource types and create a solid governance framework for your Azure subscriptions. Since you can assign Azure Policies not only to the subscriptions but to the Management groups or individual resource groups, this technique creates impressive possibilities for managing your Azure resources with greater efficiency and transparency.