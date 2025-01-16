---
title: "How to enforce naming convention for Azure resources"
seoDescription: "Learn how to enforce naming conventions for Azure resources using Azure Policy to enhance governance and improve user experience across teams"
datePublished: Tue Dec 10 2019 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5mnghmd000409mhab7b8s7r
slug: how-to-enforce-naming-convention-for-azure-resources
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737018897560/e23a201e-773b-4688-a4f3-9a6f0cde9a1d.png
tags: azure, azure-policy, naming-conventions

---

Many organizations that adopt cloud solutions do this more evolutionarily than all at once. As a result, proper governance of Azure subscriptions and services is a commonly underrated aspect of working with cloud services at the start of such transformations. At some point, an organization might end up with Azure subscriptions that contain a bunch of resources named in a completely different manner with no informative tags about resource ownership, their purpose, links to technical documentation, etc. To bring some order in this mess, people usually try to come up with a naming convention for Azure services they use and follow it to ensure a standard user experience across the teams. This approach helps to some extent, but when many teams create Azure resources, keeping all of them compliant with your naming standard becomes a real challenge.

## Using Azure Policy for naming convention

One tool that can help you enforce naming conventions for Azure resources in your subscriptions is Azure Policy. Its mechanics allow you to create controls that will validate existing and new resources for compliance.

Regarding the naming convention, you can [validate resource names against specific naming patterns and forbid the creation of resources whose names don’t satisfy pattern requirements](https://docs.microsoft.com/en-us/azure/governance/policy/samples/#naming). There are a few things to remember when using this kind of policy.

Firstly, policies for naming conventions are no different from any other Azure policies: they can be assigned at the management group level, subscription level and individual resource groups. This is especially important when testing your policy effects in your target environments—you can assign your naming policies to a test resource group and check their behaviors on a limited scope.

Secondly, if you are targeting your naming policies to the subscriptions that contain already existing but non-compliant resources, you might want to put those resources (resource groups, to be more precise) into excluded scopes in the policy assignments. If you don’t exclude those resources from your policy assignments, the “deny” policy effect will prevent you from modifying them, as on each change, the whole resource definition is processed and validated by Azure Resource Manager against all active policy rules. Depending on your requirements, you may choose a looser approach first and only audit policy violations instead of a strict “deny” policy effect.

Thirdly, in a very diverse organization with many independent teams with a high level of freedom to build and operate their products and services, you can create different naming policies for other teams and limit their assignment to specific subscriptions or resource groups. Keep in mind that naming conventions shouldn’t be enforced for the sake of policies themselves but to improve user experience and help teams perform better.

## Azure Policy support for Regex

If your policy for resource naming is quite simple, the existing [‘like/notLike’ and ‘match/notMatch’ policy conditions](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#conditions) should be enough to do the job. However, in more complex infrastructures, you might want your names to contain information about resource type, environment, version or another unique identifier, etc., so your resource name looks like the following:

`-[-][-]`

For those familiar with [regular expressions](https://en.wikipedia.org/wiki/Regular_expression), aka regex, it might be the right place to use them for naming pattern validation. Unfortunately, [Azure policy currently doesn’t support regex](https://feedback.azure.com/forums/915958-azure-governance/suggestions/34148341-azure-policy-support-for-regex-in-match-conditio), making our work harder. For instance, to implement the mentioned sample pattern, you must put a lot of copy-pasted blocks in your policy definition, and it will still be limited only to that specific list of match conditions. So, go to [feedback.azure.com](https://feedback.azure.com/forums/915958-azure-governance/suggestions/34148341-azure-policy-support-for-regex-in-match-conditio), vote for that feature, and hope that Microsoft releases it soon. Until then, a kludge with all possible naming combinations is your only option.

## Using the same Azure Policy for different resource types

Remember that there are a lot of different Azure resource types? Now, imagine that you have to define naming policies, at least for a few dozen of them in your environment. Sounds like a ton shit of work, right? So, how to make it easier?

As you might already know, Azure policy definitions support [input parameters](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#parameters), and you can use policy functions to [reference them](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#using-a-parameter-value). So, why not provide Azure resource type as an input parameter during policy assignment? There is even a corresponding ‘[strongType](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#strongtype)’ metadata property – **resourceTypes**, to validate inputs against existing resource types in your subscription:

%[https://gist.github.com/andrewmatveychuk/c8308686c9167a4a774cd2eb4046269d] 

Now, you can reference these parameters in a policy rule:

%[https://gist.github.com/andrewmatveychuk/3c0d73c8347aadf44693250f074b0a33] 

With such a policy definition, you can create multiple assignments for the same policy with different inputs: basically, for different resource types.

Of course, you might consider defining separate policies for resource types that don’t allow hyphens in their names, such as storage accounts, container registries, etc. So, consider the existing [naming rules and restrictions for Azure resources](https://docs.microsoft.com/en-us/azure/architecture/best-practices/resource-naming):  the length of resource names and valid characters vary for different resource types. So, pay attention to that.

Although it might be helpful to follow the [best practices](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging) for resource naming when designing your naming convention, you should understand that naming policies as a part of your governance framework will have to evolve with time to satisfy changing conditions. So, don’t try to make them perfect. The chances are that you might want to reconsider them soon. Likely, with Azure services, it’s much easier to spin up new copies of your environments and migrate data to them than it used to be with physical servers.

What naming convention do you prefer for your Azure resources? How do you ensure that your resources are compliant with it?

P.S. You can find sample Azure policies for naming defined in ARM templates in the following [repository on GitHub](https://github.com/andrewmatveychuk/azure.policy).