---
title: "How to deploy Azure Policies with ARM templates"
seoDescription: "Learn to deploy Azure Policies using ARM templates and automate your policy management with various tools like REST API, PowerShell, and Azure CLI"
datePublished: Tue Nov 19 2019 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5o3nuwf000009juc3owa9w4
slug: how-to-deploy-azure-policies-with-arm-templates
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922475/653b46c2-7deb-45df-a97c-bf482496f7c4.png
tags: azure, azure-policy, arm-templates

---

> This post describes an outdated experience of authoring Azure Policy with ARM templates. For a new approach, check out my post on “[How to deploy Azure Policy with Bicep](https://andrewmatveychuk.com/how-to-deploy-azure-policy-with-bicep).”

When creating custom Azure Policy definitions and assignments for them, there are a few options for doing this programmatically:

* using the [REST API](https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-and-manage#create-a-policy-definition-with-rest-api);
    
* running the [PowerShell cmdlets](https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-and-manage#create-a-policy-definition-with-powershell);
    
* executing the [Azure CLI commands](https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-and-manage#create-a-policy-definition-with-azure-cli);
    
* defining them in ARM templates.
    

Let’s give a brief overview of them.

If I were you, I would consider using [Azure REST API](https://docs.microsoft.com/en-us/rest/api/azure/) as a fallback option when there are no other ways to interact with an Azure resource. I’m not a big fan of it because I see it as a low-level interface that requires more effort to use and maintain in automation solutions.

Regarding Azure PowerShell and Azure CLI for working Azure Policies, I feel somewhat confused about their cmdlets and commands. For example, the ‘[New-AzPolicyDefinition](https://docs.microsoft.com/en-us/powershell/module/az.resources/new-azpolicydefinition)’ cmdlet requires you to provide policy rules and parameters as separate files and specify policy name, display name, description and metadata as cmdlet input parameters. So, instead of keeping all information defining an Azure Policy in one place, you must synchronize it among [multiple files](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/policy-as-code#create-and-update-policy-definitions).

Azure CLI commands resemble the same usage pattern, introducing duplication in [the contribution process](https://github.com/Azure/azure-policy/tree/master/1-contribution-guide). When using out-of-the-box tools, this makes automated deployments for Azure Policies a mixture of data and logic.

To organize your deployment pipeline, you might consider creating custom deployment scripts to [parse all required information from a single ‘azurepolicy.json’ file](https://blog.tyang.org/2019/05/19/deploying-azure-policy-definitions-via-azure-devops-part-1/) or describing your Azure Policy definitions and assignments in an ARM template, which is my choice for a consistent deployment approach in Azure.

> Note: At the time of writing, it is not possible to use ARM templates to deploy resources on a [Management Group](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview) level. Use Azure PowerShell or Azure CLI options for that.

## How to define policy definitions in ARM templates

Unfortunately, the documentation about defining Azure Policy resources in ARM templates is not very descriptive about the technical aspects of this approach: just [a short example of defining a policy definition in a template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/deploy-to-subscription#define-and-assign-policy), which, for an unknown reason, is located in a completely different section of the documentation. So, what to pay attention to?

Firstly, when creating ARM templates with Azure Policy definitions, use [the schema for subscription-level deployment](https://docs.microsoft.com/en-us/azure/azure-resource-manager/deploy-to-subscription#schema):

*https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#*

Also, if you use Azure PowerShell, use the ‘New-AzDeployment’ cmdlet instead of ‘New-AzResourceGroupDeployment’ to deploy the subscription-level templates.

Secondly, **if your custom policy definition requires** [**input parameters**](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#parameters) **or uses** [**policy functions**](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#policy-functions)**, such as ‘concat,’ for instance, use** [**escape characters**](https://docs.microsoft.com/en-us/azure/azure-resource-manager/template-expressions#escape-characters) **for them in the policy definition body** so they are not invoked during the template deployment and just passed through as parts of the policy definition:

%[https://gist.github.com/andrewmatveychuk/0ac1e9e42a05b124b89cbe66fa9a2ad9] 

Thirdly, remember that you can use the [**strongType**](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#strongtype) property to validate the input parameters of your policy.

%[https://gist.github.com/andrewmatveychuk/0641d3ba34dfad9bcb6161b203faa7d0] 

Lastly, don’t forget to specify an ‘apiVersion’ for the ‘Microsoft.Authorization/policyDefinitions’ resource provider in your template. This property is not required in the bare ‘azurepolicy.json’ format, but it is needed for Azure Resource Manager to ‘talk’ to a specific API version.

A sample Azure Policy definition in an ARM template:

%[https://gist.github.com/andrewmatveychuk/cf89d4deab2d05817d541995d057fba6] 

## How to define policy assignments in ARM templates

If you could define your Azure Policy definition in an ARM template and successfully deploy it, then [creating a policy assignment](https://docs.microsoft.com/en-us/azure/governance/policy/assign-policy-template#create-a-policy-assignment) will be a piece of cake. There is no need to use escape characters for policy definitions – policy assignments can be treated as regular ARM resources. However, to pass the policy parameters, you should define them [as an object](https://docs.microsoft.com/en-us/azure/azure-resource-manager/deploy-to-subscription#assign-policy) input both in the template and parameter file (see ‘[Use an object as a parameter in an Azure Resource Manager template](https://learn.microsoft.com/en-us/azure/architecture/guide/azure-resource-manager/advanced-templates/objects-as-parameters)’ for details):

%[https://gist.github.com/andrewmatveychuk/d40cafc69a87cf8353a243d8587be07a] 

Also, there is one crucial aspect to be aware of. When creating Azure Policy assignments on the portal, you can limit the assignment scope to a specific resource group, which might be especially handy for testing policy effects before deploying it to production. To achieve the same effect in an ARM template, you can specify the ‘scope’ property in the following format:

*/subscriptions/&lt;subscription\_id&gt;/resourceGroups/&lt;resourcegroup\_name&gt;*

and stumble at the following non-descriptive error:

> *“The policy assignment &lt;policy\_definition\_name&gt; create request is invalid. Policy assignment scope ‘/subscriptions/&lt;subscription\_id&gt;/resourceGroups/&lt;resourcegroup\_name&gt;’ must match the scope specified on the Uri ‘/subscriptions/&lt;subscription\_id&gt;’.”*

If you expand the scope to the subscription level, the deployment will be compiled without errors.

So, the trick here is to:

1. Use a [regular deployment schema](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates#template-format) like ‘[https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#’](https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#%E2%80%99),
    
2. Deploy the ARM template with policy assignment using the ‘New-AzResourceGroupDeployment’ cmdlet when scoping for a specific resource group and ‘New-AzDeployment’ when targeting a subscription.
    

In terms of automation, you can create a single unified deployment template that takes policy definition and its required parameters as inputs and separate parameter files for each policy assignment:

%[https://gist.github.com/andrewmatveychuk/4fa839a8e78cf1cffc8add238dfa80e1] 

## Deployment scripts for Azure Policy definitions and assignments

Now, when you have your ARM templates for policy definitions and assignments ready, it is time to deploy them.

Even though it is possible to [define Azure Policy definitions and assignments in the same template file](https://docs.microsoft.com/en-us/azure/azure-resource-manager/deploy-to-subscription#define-and-assign-policy), I prefer to make the deployment a two-step process: deploy policy definitions first, then create assignments for them. The reason for that is flexibility. I created a policy definition on the subscription level but want to limit an assignment to a resource group.

If you structure your repository for policy definitions according to [the official recommendations](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/policy-as-code#create-and-update-policy-definitions), you might use the following Azure PowerShell script to automate their deployment:

%[https://gist.github.com/andrewmatveychuk/a5a1eb1187cc7789e0c6a5145fa3c40c] 

Regarding the deploying templates with policy assignment, you might consider a more dynamic approach:

%[https://gist.github.com/andrewmatveychuk/f7a69bc79eec6cb33f172b3ce3ee3d4c] 

Depending on the environment in which you are creating Azure Policy assignments, you can construct input parameters for the sample ARM template mentioned in the previous section on the fly. The Azure PowerShell deployment cmdlets can take hash tables as template parameter inputs, so you can specify your environment-specific parameters as Azure DevOps pipeline variables and add them during the deployment. Doing so can help you to reduce the number of duplicate parameter files for different environments in your repository.

Have you tried to create a deployment pipeline for Azure Policies? Share your experience in the comments!