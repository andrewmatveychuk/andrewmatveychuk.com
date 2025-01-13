---
title: "Using ARM templates to deploy Azure Policy initiatives"
seoDescription: "Learn how to deploy Azure Policy initiatives using ARM templates for efficient governance and bulk policy assignments"
datePublished: Tue Dec 03 2019 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5monkki000209mn7ayt14s0
slug: using-arm-templates-to-deploy-azure-policy-initiatives
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922863/d31ba1c7-785d-4c6e-91e9-aa64f55c2c73.png
tags: azure, azure-policy, arm-templates

---

In my [previous blog post](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates), I explored the topic of defining Azure Policies and their assignments in ARM templates, so instead of maintaining lots of separate files for policy definitions, you can put all policy-defining artifacts in an ARM template and deploy it in a pipeline along with other parts of your Azure infrastructure. However, in some cases, it might be more convenient to group the policies targeting a specific area of your Azure governance model and assign them to a subscription in bulk. So, here, [Azure Policy initiatives](https://docs.microsoft.com/en-us/azure/governance/policy/overview#initiative-definition) come to help.

You can think about Azure Policy initiatives just as collections of policy definitions, allowing you to assign all the policies in a collection in a [single strike](https://docs.microsoft.com/en-us/azure/governance/policy/overview#initiative-assignment).

> Note: At the time of writing this, the same limitations apply for working with policy initiatives as for single policy definitions – it is not possible to use ARM templates to deploy resources on a Management Group level. Use Azure PowerShell or Azure CLI options for that.

## How to define Azure Policy initiatives in ARM templates

As with policy definitions, there is even less information or descriptive examples of defining policy initiatives in ARM templates. You can only get more or less relevant information from [sample responses to Azure REST API requests](https://learn.microsoft.com/en-us/rest/api/policy/policy-definitions/create-or-update). According to these examples, you can use the same approach when describing initiative definitions in the ARM template for single policies, but with a few differences.

Firstly, the ARM template’s structure remains the same: you use [the same schema reference for subscription-level deployments](https://docs.microsoft.com/en-us/azure/azure-resource-manager/deploy-to-subscription#schema) and the same high-level blocks — parameters, variables, resources, etc.

Secondly, as initiatives are sets of policies, you should use the ‘**Microsoft.Authorization/policySetDefinitions**’ resource type to define your initiative.

Thirdly, when describing initiatives, you don’t define policies in them but rather reference existing ones. Of course, you can put both policy and initiative definitions in a single ARM template, but to deploy it correctly, you should specify dependencies so that policies referenced in the initiatives are deployed first.

So, a sample initiative definition might look like in the following example:

%[https://gist.github.com/andrewmatveychuk/930d3981823d5f7809312506044c64a4] 

## How to create assignments for Azure Policy initiatives from ARM templates

As well as for individual Azure Policies, you can also define assignments for initiatives in ARM templates. There is no separate resource type for initiative assignments, so you should use the already familiar ‘Microsoft.Authorization/policyAssignments’ type. The only difference is that to assign an initiative to a target scope, you should reference ‘Microsoft.Authorization/policySetDefinitions’ in the ‘policyDefinitionId’ property.

Also, if you plan to assign policy initiatives to the subscription level and individual resource groups, you should [use the regular deployment schema in your ARM template and deploy it using resource group deployment cmdlets](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates).

Here is a sample fragment of the ARM template for the policy initiative assignment:

%[https://gist.github.com/andrewmatveychuk/ead6ea0b3849b2664391814b686a31ec] 

## Azure policy parameters as object type when assigning an initiative

As you might already know from my [previous post](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates), if an Azure Policy takes input parameters, you should provide them as an object type during the assignment. Because the deployment order looks like the following:

*Parameter file for an ARM template -&gt; an ARM template with an Initiative assignment -&gt; an Initiative definition -&gt; (policy parameters as an object) -&gt; a Policy definition*

Policy parameters must be defined as objects in initiative definitions and passed as objects during initiative assignments. To achieve that, you should pack the parameter objects into another (parent) object in the parameter file so the parameter file has the following structure:

%[https://gist.github.com/andrewmatveychuk/d0fdee32ae9e8d997425cf4b7ff48e44] 

It took me some trial and error to figure out that nuance, as error messages just tell you about some JSON parsing or referencing issues without any hints about processing parameter values.

I hope you will find this information helpful on your journey through Azure services. Please like it and share it with your colleagues!

P.S. You can find all code samples from this post in the following [repository on GitHub](https://github.com/andrewmatveychuk/azure.policy).