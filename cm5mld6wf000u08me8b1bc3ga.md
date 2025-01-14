---
title: "Audit and Enable Azure Hybrid Benefit using Azure Policy"
seoDescription: "Optimize Azure cost by auditing and enabling Azure Hybrid Benefit with Azure Policy for better resource compliance and management"
datePublished: Tue Nov 29 2022 13:48:26 GMT+0000 (Coordinated Universal Time)
cuid: cm5mld6wf000u08me8b1bc3ga
slug: audit-and-enable-azure-hybrid-benefit-using-azure-policy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736871029044/ca5e63db-1e21-43f6-a4c2-666ea8b84f41.png
tags: azure, azure-hybrid-benefit, azure-policy

---

## What is Azure Hybrid Benefit?

From the cost perspective, the resulting price for an Azure resource is calculated from multiple parts, software licenses being one of them. For example, when using the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/) and estimating your Azure VM cost, you might notice that the total row changes depending on the operating system and VM configuration type (OS only, SQL Server, etc.). For some of that software, you can apply [Azure Hybrid Benefit](https://azure.microsoft.com/en-us/pricing/hybrid-benefit/), which basically allows you to bring your own license (BYOL, for short) and save on your cloud infrastructure spend in Azure.

> Speaking of Microsoft software, it’s a great cost optimization case for enterprise customers that usually already have lots of Windows and SQL Server licenses as part of their Enterprise agreements. That is especially true for customers with a significant portion of their existing infrastructure running outside of Azure, presumably in their on-premises data centers, and migrating their workloads to Azure.

Different Microsoft products have different conditions and applicability logic for Azure Hybrid Benefit, and I will cover those details in the following sections.

## What is Azure Policy?

[Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/overview) is an Azure service that can be used to “implement governance for resource consistency, regulatory compliance, security, cost, and management.” In other words, it’s a framework that allows you to define rules for resource configuration, audit resource compliance with those rules, and enforce the rules by preventing the deployment of non-compliant resources or modifying existing resources so they become compliant with them.

That service is well-documented, so I’m not going into the details about how it works here. If you need a quick overview, you can check my [Azure Policy Starter Guide](https://andrewmatveychuk.com/azure-policy-starter-guide) and [Azure Policy-related content](https://andrewmatveychuk.com/tag/azure-policy).

The question worth asking here is whether Azure Policy is a good tool for our purpose, which is auditing and enabling Azure Hybrid Benefit. Firstly, as you can see from the service definition, it is specifically designed for such tasks. Secondly, it implements a declarative approach for configuration description. Lastly, Azure Policy provides all necessary tools for reporting, decoupling policy definitions and assignments, and enforcing resource configuration. So, let’s consider some practical examples.

## Azure Hybrid Benefit for Windows Server VMs

[Azure Hybrid Benefit applicability for Windows VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing) is controlled by the **licenseType** property of an Azure VM. Basically, you can switch it on or off by updating the license type to ‘*Windows\_Server*’ or ‘*None*’. You don’t even need to restart the VM for that, as it is a feature of Azure Compute service and not an operating system.

The Azure Policy rule that defines the applicability logic of that benefit to your Azure VMs may vary depending on how you define your eligible VM instances. Generally, you can use specific OS images as a filtering condition, as Windows Server licenses are usually related to specific OS versions. That way, you can define the list of the benefit-eligible OS images as an input parameter and update it dynamically as your license term changes over time. Sample policy rule to enumerate eligible Windows Server Azure VMs in a policy assignment scope might look as follows:

%[https://gist.github.com/andrewmatveychuk/90fb0568ca12fed0f9b8c0074ab98d6d] 

You might notice that the policy effect is also defined as a parameter here. I highly suggest using such an approach so you can change your policy assignment effect without editing the policy definition and go from auditing to preventing (denying) the deployment of eligible Azure VMs without Azure Hybrid Benefit enabled in a target scope.

Similarly, you can craft an Azure Policy to enroll the eligible Azure VMs into using the licensing benefit:

%[https://gist.github.com/andrewmatveychuk/80e2b2406bf963fad996f6c1704eb7d4] 

You can find the complete policy definitions in [my Azure Policy GitHub repository](https://github.com/andrewmatveychuk/azure.policy/tree/master/other-samples/policies/definitions/azure-hybrid-benefit). As I prefer using Bicep to write my Azure deployment templates, you might want to check my other post on [how to deploy Azure Policy with Bicep](https://andrewmatveychuk.com/how-to-deploy-azure-policy-with-bicep).

## Azure Hybrid Benefit for Windows Client VMs

The same Azure VM property controls Azure Hybrid Benefit for Windows Client VMs, but it’s called [Multitenant Hosting Rights](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/windows-desktop-multitenant-hosting-deployment#subscription-licenses-that-qualify-for-multitenant-hosting-rights), allowing you to use your existing Windows 10/11 licenses in Virtual Desktop scenarios. To apply the benefit for Azure VMs running the desktop OS, the license type should be set to ‘*Windows\_Client*.’

From the Azure Policy perspective, you can use the same filtering approach as with Windows Server Azure VMs but target Windows Client OS images specifically:

%[https://gist.github.com/andrewmatveychuk/7e4e102b84b30f416cdb886d4fa6cac2] 

Just pay attention to a different image publisher and another license type here.

With virtual desktop scenarios like Azure Virtual Desktop, especially in the personal host pools scenario, it’s even more important to automate the configuration of provisioned hosts with enabled Multitenant Hosting Rights to reduce your AVD costs if they are eligible to use that benefit, which is a typical case for customers having [Microsoft 365 E3, E5 and other entitlement licenses](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/windows-desktop-multitenant-hosting-deployment#license-entitlement). You can achieve this on the fly when a new personal AVD host is provisioned using the Modify effect of Azure Policy.

[My GitHub repository](https://github.com/andrewmatveychuk/azure.policy/tree/master/other-samples/policies/definitions/azure-hybrid-benefit) contains the complete policy definitions to audit and enable Azure Hybrid Benefit for Windows Client VMs.

## Azure Hybrid Benefit for SQL Server VMs

[SQL virtual machines in Azure can utilize your SQL Server licensed cores](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change) to achieve a similar licensing benefit.

> Note that Azure Hybrid Benefit applies to Standard and Enterprise SQL Server editions only.

From the technical point of view, your SQL Server VMs must be appropriately registered with [the SQL IaaS Agent Extension](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management?view=azuresql). Also, the applicability of the benefit is controlled by the **sqlServerLicenseType** property of the SQL Server VM resource. It should be set to:

* AHUB for the Azure Hybrid Benefit
    
* PAYG for pay-as-you-go
    
* DR to activate the free HA/DR replica
    

A sample Azure Policy rule to evaluate SQL Server VMs for Azure Hybrid Benefit usage can be as follows:

%[https://gist.github.com/andrewmatveychuk/7b0b3f3b922b1724985371d6330a0d5d] 

> Interestingly, you can apply Azure Hybrid Benefit for an underlying Windows Server VM and SQL Server instance running on it, provided that you have all corresponding licenses, archiving even more substantial savings. So, consider that when estimating potential Windows/SQL hosting costs in Azure.

Suppose you want to logically combine your configuration rules (policies) for both Windows Server and SQL Server licensing benefits to apply them at once. In that case, I suggest using [Policy Initiatives](https://andrewmatveychuk.com/using-arm-templates-to-deploy-azure-policy-initiatives), aka Policy Definition Sets. Such a solution will be more flexible and modular to manage.

For a complete policy definition, check [my GitHub repository](https://github.com/andrewmatveychuk/azure.policy/tree/master/other-samples/policies/definitions/azure-hybrid-benefit).

## In conclusion

Technically speaking, similar configurations can be enforced using the corresponding parameters in Bicep definitions for your resources or running an Azure PowerShell one-liner to modify resource properties. However, the primary disadvantage of those approaches is that they require additional effort to prevent configuration drift and ensure compliance at scale. Having all your infrastructure defined in code (IaC) is good, but being able to test and validate it for compliance is even better. In the case of Azure PowerShell (or Azure CLI), manually running a script is not enough to ensure resource compliance in an ever-changing environment with thousands of resources. You need to schedule regular jobs to validate resource configuration, report on non-compliant resources, and develop a mitigation strategy.

Similarly, Azure Policy is not a silver bullet, as it also has limitations. For example, to comply with your license agreements, you should keep track of the number of applied Azure Hybrid benefits and how they convert to the number of licensed OS and SQL Server cores. Implementing such logic with Azure Policy seems overcomplicated as it wasn’t intended for such tasks. In one of the upcoming posts, I plan to cover how you can achieve that using Azure Monitor Workbooks. If you don’t want to miss that update, please subscribe to my new posts using the form below 👇