---
title: "How to deploy Azure Policy with Bicep"
seoDescription: "Learn how to deploy custom Azure policies using Bicep with tips on policy, initiative assignments, and advanced techniques"
datePublished: Wed Apr 28 2021 12:00:18 GMT+0000 (Coordinated Universal Time)
cuid: cm5mo2grk000709la3enwegug
slug: how-to-deploy-azure-policy-with-bicep
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922823/bf0d9318-b372-41bc-a3f1-d675766666b5.png
tags: azure, bicep, azure-policy

---

It has been a while since I last wrote about [Azure Policy](https://andrewmatveychuk.com/tag/azure-policy/), and recently, there was a lot of hype around [Bicep](https://github.com/Azure/bicep), so I decided to give it a try and shed some light on creating and deploying custom Azure policies with that new language.

## Prerequisites

I assume that you are already familiar with [Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview) and how it works. If you are new to that really helpful and often underrated technology, I suggest checking out my [Azure Policy Starter Guide](https://andrewmatveychuk.com/azure-policy-starter-guide).

Also, I recommend that you read through [the official Bicep documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/bicep-tutorial-create-first-bicep) to get a sense of this new domain-specific language, which Microsoft promotes as an abstraction over ARM templates and Azure Resource Manager.

## Bicep basics for Azure Policy

Like JSON-based ARM templates, Bicep is a declarative language that allows you to define the desired Azure resource configuration and let the Azure Resource Manager provision it. Initially, you had to compile a Bicep file into a regular ARM template to deploy your configuration. However, this is not the case anymore, as both Azure CLI and Azure PowerShell now support deploying Bicep definitions. Note that input parameters for Bicep definitions still come in [the same format as for old-school ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/bicep-tutorial-use-parameter-file?#add-parameter-files).

Apart from that, as we work with the same Azure service, you should expect that all Azure Policy specifics are applicable regardless of the language you use to define your configurations. So, [all the tips and tricks you learned about creating, deploying and evaluating Azure Policy](https://andrewmatveychuk.com/tag/azure-policy/) are still relevant.

Speaking of Bicep, you can define a single policy and a policy initiative, aka policy set, along with their assignment to a specific scope using the [Bicep resource primitive](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/resource-declaration). For example, to create a custom Azure Policy, you can define the following resource in your Bicep file:

%[https://gist.github.com/andrewmatveychuk/19c66809dd213575aab9c3195a225b7e] 

As you might notice from that sample, for some policy-specific syntax to be valid, you should use backslash as an escape character for single quotation marks. Also, you shouldn’t use an [additional forward square bracket in the expressions](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates), as it will be automatically added to the JSON during the Bicep build.

In the same manner, you can define your custom policy initiative:

%[https://gist.github.com/andrewmatveychuk/fb891fa9162675190c3d2299d41f9445] 

Policy initiatives don’t define any rules, so they use the ‘**policyDefinitions’** keyword to reference existing policy definitions.

Policy and policy initiative assignments are also pretty straightforward and defined as yet another resource:

%[https://gist.github.com/andrewmatveychuk/41c3a9bc9ab58118959b290a8d7b0b38] 

For complete definitions, look into the bicep samples in [my Azure Policy repository on GitHub](https://github.com/andrewmatveychuk/azure.policy/). The Bicep product team and the community regularly update and create new [sample Bicep definitions](https://github.com/Azure/bicep/tree/main/docs/examples) for various Azure services, including Azure Policy, so I suggest checking them for additional cases.

## Advanced technics

Now, when the basics are clear, let’s look into more advanced topics.

First of all, remember that you can only deploy your custom Azure policy definitions at [the subscription and management group levels](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#definition-location). At the same time, Azure Policy assignments can be created at the management group, subscription, and resource group levels. Also, keep in mind that the deployment scope of a policy, in turn, effectively limits its [assignment scope](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/scope).

> The [Bicep VS Code extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep) will warn you about the resources that cannot be deployed to the target scope, and the Bicep compiler will produce the compilation error.

Regarding Bicep definitions, you can scope your deployments by using the ‘[targetScope](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions-scope)’ keyword. Depending on that scope, a bicep file will be compiled into an ARM template using the corresponding deployment schema.

Using different deployment scopes will also impact how you reference other resources in your configuration. Using standard syntax like ‘[resourceSymbolicName.Id](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions-resource#resourceid)’ should be enough if a new resource is defined in the same Bicep file. However, when you need to reference an [existing resource](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/existing-resource), e.g., a policy definition in policy initiative, first, you should correctly define that resource in your Bicep file, and second, you should use the correct reference function:

* for Azure Policy definitions deployed at the subscription level, use the ‘[subscriptionResourceId](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-resource?tabs=bicep#subscriptionresourceid-example)’ function;
    
* for Azure Policy definitions deployed at the management group level, use the ‘[extensionResourceId](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-resource?tabs=bicep#extensionresourceid-example)’ function as custom policy definitions are implemented as [resource extensions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/extension-resource-types#microsoftauthorization).
    

> If you want to assign your policy at the tenant level, you should use the [Tenant Root Group](https://docs.microsoft.com/en-us/azure/security-center/security-center-management-groups#introduction-to-management-groups) for that.

For example, to reference your existing policy definition deployed at the subscription level in a policy initiative, you can define it as the following:

%[https://gist.github.com/andrewmatveychuk/3788ef70e1fc1832854eac13fc542222] 

Same referencing but at the management group level can be accomplished with the following syntax:

%[https://gist.github.com/andrewmatveychuk/9f4926e8d1bb8a267e47b64f6616985e] 

> I don’t cover the Azure Policy exemptions feature here as it’s currently in preview and might change in the future.

## Current drawbacks

Unfortunately, authoring Azure Policies with Bicep is still far from ideal. So, here are a few things that annoyed me, not specifically related to Azure Policy but rather Bicep language-generic features or their absence.

As I already mentioned, [referencing policies from policy initiatives](https://github.com/Azure/bicep/issues/1228) is a bit complicated and non-intuitive, as you must be explicit about your reference scopes and always keep that nuance in mind. On a larger scale, when you have dozens of definitions that are defined in separate files and need to be deployed and assigned at different scopes, the new authoring experience is quite painful.

The next thing that adds up to the negative authoring and debugging experience is [the lack of IntelliSense support for Azure Policy](https://github.com/Azure/bicep/issues/1720) internal logic defined in Bicep files. Moreover, the syntax highlighting for Bicep-defined Azure Policy rules in VS Code is also very limited. Apart from that, I was also unpleasantly surprised that, in contrast to the ARM template authoring experience, [Bicep will not warn you about unused parameters or variables](https://github.com/Azure/bicep/issues/1949) you have in your files.

Lastly, the way you currently define human-readable names and descriptions for policy parameters using the [Bicep parameter decorators](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/parameters#decorators) looks a bit awkward to me:

%[https://gist.github.com/andrewmatveychuk/dc25eb910982b00e062283d533c69a40] 

Why not provide [the same simplified syntax for the ‘name’ metadata tag for the ‘description’](https://github.com/Azure/bicep/issues/2455)?

Fortunately, the [Bicep product team](https://github.com/Azure/bicep/graphs/contributors) is progressing well in implementing new features and providing the community with simpler and better options for defining Azure infrastructure as code. I’m impatiently looking forward to new [Bicep releases](https://github.com/Azure/bicep/releases).

If you are writing lots of ARM templates and haven’t tried Bicep yet, I recommend you do so and post your impressions in the comments below!