---
title: "Azure Policy: Starter Guide"
seoDescription: "Azure Policy starter guide: best practices, resources, creation, assessment, testing, deployment for effective cloud governance"
datePublished: Mon Feb 01 2021 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b5ga9z000i08js3kh80zci
slug: azure-policy-starter-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737043067044/15e5c87f-00eb-4699-ac3c-c60abb5cbb05.png
tags: azure, how-to, cloud-governance, azure-policy

---

My coworkers and teammates often reach out to me with similar questions regarding the best practices for creating and applying [Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview). That tendency encouraged me to compile this starter guide for Azure Policy, which is based on my practical experience in multiple projects and covers the 20% baseline that allows you to implement 80% of typical use cases, aka [the Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle#In_computing).

## Learn the topic

*RTFM stands for “read the fucking manual,” bro.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922536/3d3fe24b-4581-48ae-a837-0b63076b937d.png align="center")

Seriously, I mean, read the [Microsoft Azure Policy docs](https://docs.microsoft.com/en-us/azure/governance/policy/) first. Microsoft has been doing a great job documenting its services and solutions recently, and without knowing the basic Azure Policy principles, it will be really hard for you to grasp the concepts. After investigating what Azure Policy is for, I suggest looking through [the list of built-in policies](https://docs.microsoft.com/en-us/azure/governance/policy/samples/built-in-policies) to get an idea about typical use cases for different Azure service types.

The two most important points to pay attention to initially are [understanding Azure Policy effects](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects) and [Azure Policy deployment scopes](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/scope). The effects will give you some insights into what you can actually do with the policies. At the same time, the deployment scope will save you time for troubleshooting why you cannot assign a policy deployed at the subscription level to another subscription.

I would say the evaluation of logical conditions in policy rules is less critical. It might cause you some headaches initially, but as soon as you understand how the double negation works, you will be fine.

> Although Azure policies can modify the configuration of existing Azure resources and even deploy new resources, I suggest starting with auditing resource configuration (Audit and AuditIfNotExists effects) and putting some guardrails (Deny effect) in your environment as the latter ones are easier to learn and understand.

Apart from the official documentation, I definitely recommend watching a few learning courses about Azure Governance on [Pluralsight](https://andrewmatveychuk.com/refer/pluralsight):

* [Mastering Microsoft Azure Governance by James Bannan](https://andrewmatveychuk.com/refer/microsoft-azure-governance-mastering)
    
* [Microsoft Azure DevOps Engineer: Implementing Infrastructure Control and Compliance by John Savill](https://andrewmatveychuk.com/refer/microsoft-azure-infrastructure-control-compliance-implementing)
    

They are just a few hours long and can provide a good starting point for advancing your Azure Policy learning.

## Make an assessment

*“Think first before you act.” An unknown guru.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923377/b1bb23fb-09d2-4d60-aeb4-aede83644cbf.png align="center")

Before making any changes in your environment, i.e., assigning a new Azure Policy to your subscription, it is worth knowing first what policies are already in effect and their compliance results. Besides, [assigned policies are evaluated in a specific order](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#order-of-evaluation) you should know. Otherwise, it is easy to mess up your Azure environment: policies usually control something on a global scale (a whole subscription or management group), therefore impacting lots of resources.

In 80 percent of use cases, [using the Azure portal to assess](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#portal) existing policy and initiative assignments and their compliance state will be the right choice - when only a few policies are applied, there is no need to overcomplicate things.

In more advanced scenarios, when an organization already deployed dozens of custom Azure Policy definitions and extensively uses them at the management group level and on the individual subscriptions, manual assessment is somewhat complicated. Here, I can suggest using [AzGovViz](https://github.com/JulianHayward/Azure-MG-Sub-Governance-Reporting)– a community build solution (a PowerShell script) that can help you quickly create a comprehensive report in different formats containing all the details about Azure Policy configuration in your environment and more. You can even integrate [AzGovViz with Azure DevOps](https://jacktracey.co.uk/azgovviz-with-azure-devops/) pipelines to document the policy configuration in your deployments.

## Create your policy

*“Let’s roll up your sleeves and get to work!” A motivational speech.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923959/af15e198-8b8e-4585-899d-9e3e0f69fb55.png align="center")

Even though Microsoft has already provided us with many useful built-in ready-to-use policies, I encourage you not to hurry on assigning them left and right. You will never understand how Azure Policy works to the full extent until you learn how to create and manage your custom policies.

> A typical antipattern to avoid is dozens of individually assigned policies when they should be applied as a group via a policy initiative.

Firstly, you can look into the source code of [built-in Azure policies](https://docs.microsoft.com/en-us/azure/governance/policy/samples/built-in-policies) (check the last column with the links to GitHub) and use it as a draft for your custom policy or initiative definitions. Alternatively, you can go straight to the [Azure Policy Samples repository on GitHub](https://github.com/Azure/azure-policy), clone it, and explore it with your coding tools.

The best coding experience with Azure Policy as of now is probably [using Visual Studio Code with the Azure Policy extension](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/extension-for-vscode). Additionally, I suggest installing [the ARM Tools extension](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools). It will significantly help you with syntax validation, snippets, and auto-completion if you decide to [define your policies in ARM templates](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates) to make your deployment experience more consistent.

> Recently, Microsoft has updated its docs with some ARM snippets for [policy definitions](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policydefinitions), [policy set definitions](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policysetdefinitions) (aka policy initiatives), and [their assignments](https://docs.microsoft.com/en-us/azure/templates/microsoft.authorization/policyassignments). Still, those articles miss many nuances and details, and I suggest checking out [my work on Azure Policies](https://andrewmatveychuk.com/tag/azure-policy) and [my repository for sample Azure Policies on GitHub](https://github.com/andrewmatveychuk/azure.policy).

For more advanced cases, check [the recent updates to Azure Policy on AzPolicyAdvertizer](https://www.azadvertizer.net/azpolicyadvertizer_history.html). As documenting new policies usually takes some time, [AzPolicyAdvertizer](https://www.azadvertizer.net/azpolicyadvertizer_history.html) closes that gap by providing short information about policies and recent changes to them.

> A common use case is to duplicate a built-in policy logic in your custom definition completely. The reason for that is the way how Azure Policy engine handles updates to the existing definitions. When you update a definition, all existing policy assignment of it will automatically be using the new definition. Although there are some controls for backward compatibility, and Microsoft usually doesn’t introduce breaking changes in the existing definitions, many teams prefer to have full control over their configuration.

## Test it

*“Damn it! I said, test it first!” A senior developer is fixing a bug in production.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924615/1688d097-8637-4d33-a261-0173949a622c.png align="center")

I honestly must warn you that testing Azure Policy is not an easy task. Nevertheless, I strongly encourage you to test your policy work before using it. Considering the usual scope to which policies are applied and the effects they can have (change configuration, deploy new resources), the results of careless policy assignments can be quite devastating to your environment.

First of all, you need to ensure that the syntax of your policy or initiative is correct. Whether you define your definitions in JSON policy format or ARM templates, the Visual Studio Code extensions mentioned above should help you find and fix basic syntax errors. If you stick with [the ARM template option](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates), you can use Test-Az\*Deployment Azure PowerShell cmdlets to validate your templates’ syntax against Azure Resource Manager APIs. Unfortunately, [the policy-related cmdlets in the Az.Resources module](https://docs.microsoft.com/en-us/powershell/module/az.resources/#policy) don’t support any testing options yet.

> As a matter of caution, set the [policy ‘enforcementMode’](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/policy-as-code#test-and-validate-the-updated-definition) parameter into the disabled state when creating assignments for your tested policies so you can safely audit their work results.

Secondly, be aware that Azure Policy assignments don’t take effect immediately. There is a [policy evaluation delay](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers), which is around 30 minutes or so. Also, auditing your resources might take some time, as the Azure Policy engine needs to evaluate all resources against policy rules within the assigned scope. In other words, you cannot test the results of your policy work immediately. Apart from that, the delay effectively complicates automated tests for Azure Policy.

> Although there is an option to initiate an [on-demand evaluation scan](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#on-demand-evaluation-scan), it still won’t make the whole process much faster if a policy needs to process thousands of resources.

Due to all the complications, I would say that the testing process for your policies will be manual or semi-manual in most cases. You will validate the syntax, deploy the definitions into a test environment, i.e., a dedicated subscription, assign them to a test scope, deploy some resources to test the expected policy behavior and [check results on the portal](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#portal). In the end, the code for Azure Policy is not often updated, and manual testing can be a reasonable tradeoff for creating automated test cases.

However, in advanced scenarios, when you need to create and maintain more than a handful of simple policies, creating automated Azure Policy tests as part of your [CI/CD pipeline](https://andrewmatveychuk.com/how-to-deploy-azure-policy-from-an-azure-devops-pipeline) is a must. I plan to cover this topic in detail in a separate post as it requires quite a lot of explanation not explicitly tied to Azure Policy.

## Deploy

*“Do. Or do not. There is no try.” Master Yoda to young Skywalker.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925671/2418a4f8-21fa-46d1-bf12-b0aff2d9fb9a.png align="center")

As I already mentioned, before actually deploying your custom policy or initiative definitions, you should clearly understand what [the deployment scopes](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/scope#definition-location) are. Besides, you should also understand how [Azure Policy inclusions, exclusions, and exemptions](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/scope#assignment-scopes) work. Apart from that, you should clearly distinguish between a policy/initiative definition and its assignment: you should deploy the definition **and** assign it to your scope to make your Azure policy work.

Technically, you can deploy policies and create assignments using any supported method: [the portal, Azure CLI, Azure PowerShell, Azure REST API, etc](https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-and-manage). It’s really up to you to choose which one of them fits your configuration management and deployment practices.

When I started working with Azure Policy myself, I was a bit frustrated with the default programming experience of maintaining two separate files for each definition, so I came up with a solution on [how to deploy Azure Policy with ARM templates](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates). However, things have changed since then, and now [the policies are defined in a single file](https://github.com/Azure/azure-policy). A slight improvement, but the [Azure PowerShell cmdlets](https://docs.microsoft.com/en-us/powershell/module/az.resources/#policy) still require lots of additional parameters that should be duplicated on their usage.

Optionally, you can try using the [AzOps](https://github.com/Azure/AzOps) deployment framework, which could be a good choice for large environments when you run your Azure Governance as a separate project.

Be consistent in your deployments and preferably manage Azure Policy as [part of your CI/CD pipelines](https://andrewmatveychuk.com/how-to-deploy-azure-policy-from-an-azure-devops-pipeline).

## Check the results

*“A man reaps what he sows” A proverb.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681926343/bcb523f8-fd1e-404a-b5fa-27cd2d45d3b3.png align="center")

Finally, your first policy is deployed, the assignment is created, and it’s time to see what we have got.

> Remember about [the time it takes for policy to come into effect and evaluate your resources](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

[Using the Azure portal to get Azure Policy compliance results](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#portal) would be the most obvious and probably the most reasonable choice at the beginning – it won’t hurt to keep things simple.

For advanced scenarios, when you are already proficient with managing Azure Policy from deployment pipelines, you might want to check how to [get Policy insights with code](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#azure-powershell) to evaluate them in your test cases. Also, take a look at the [Az.PolicyInsights](https://docs.microsoft.com/en-us/powershell/module/az.policyinsights/) PowerShell module, and what kind of data you can extract with it.

## In conclusion

Just reading this guide won’t make you an expert in Azure Policy. For that, you need to have some practice too. So, give it a try – look into your Azure infrastructure, find some areas you can improve with Azure Policy (trust me, there is always something that can be improved 😉), come up with a solution, test it, apply and reap the benefits!

If you have any questions about this topic, put them in the comments below 👇.

Cheers!