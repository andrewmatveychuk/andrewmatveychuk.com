---
title: "Automatic tagging for Azure resources"
datePublished: Thu Dec 26 2019 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b9ddri000309kt2chl0wl5
slug: automatic-tagging-for-azure-resources
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737018761134/343356b0-39cd-4158-8e38-b82216907989.png
tags: azure, azure-policy

---

To tag or not to tag is out of the question. The question is how to apply tags to Azure resources only when needed and do that efficiently.

[Azure tags](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags) are just metadata information about resources, which allows you to classify and logically group them. Tags can be created via [ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/template-tutorial-add-tags?tabs=azure-powershell#add-tags), [Azure PowerShell](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags#powershell), [Azure CLI](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags#azure-cli), [REST API](https://docs.microsoft.com/en-us/rest/api/resources/tags) and [the portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags#portal), of course. Almost all [resource types support tags](https://docs.microsoft.com/en-us/azure/azure-resource-manager/tag-support). However, you should take into account some generic [limitations](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags#limitations).

## Common issues with Azure metadata tags

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922439/71b55cfd-7c91-4d53-bd0a-88a9b5903480.png align="center")

The most common issues with using Azure Tags are either not using them at all or going from one extreme to another – applying dozens of them to every resource. In the first case, you are missing an opportunity to organize and manage your resources more effectively. In the second, you add unnecessary overhead for managing all those tags that don’t provide you with any real value.

To make it worse, some organizations [enforce mandatory tagging](https://docs.microsoft.com/en-us/azure/governance/policy/samples/enforce-tag-value) for all Azure resources, so you cannot create or modify an existing resource if it doesn’t have a specific tag(s). Sure, there should be some governance for your Azure subscriptions that keeps your infrastructure neat and structured, but when it goes to an extreme, you might need to spend hundreds of additional hours modifying your ARM templates or deployment scripts to have these mandatory tags encoded in them.

When people don’t see any value for them in doing some dumb job like adding those 50+ tags to their code, they tend to look for shortcuts. For example, copy and paste the same code fragment with tags to all files or autogenerate tags with some fake values. In the meantime, you might be unpleasantly surprised that most resources in your subscription(s) have dummy tags that don’t help you to operate those resources in any way but obscure really valuable information.

## All about trade-offs

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923384/7d845281-76b4-4b67-b127-78b7dac7455f.png align="center")

The key thing to remember is that there is no single best solution for all possible cases. Each tagging strategy has its pros and cons, so you have to make trade-offs between simplicity and accuracy.

For example, one balanced approach is to demand appropriate tagging at [the resource group level](https://docs.microsoft.com/en-us/azure/governance/policy/samples/enforce-tag-on-resource-groups). As resource groups serve as a logical boundary for a group of resources that share a common life cycle, you can apply tags to these logical entities rather than to individual resources. Implementing such a tagging technique will allow you to simplify your governance without introducing excessive controls.

For cases when it is more convenient to have tags at the resource level, you can implement automatic tag copying – basically, [inherit tags from the resource group level](https://github.com/Azure/azure-policy/tree/master/samples/Tags/inherit-resourcegroup-tag). Of course, in some scenarios, tags might have meaning only for a specific resource type, and you have to choose whether to tag a resource group and copy the tag to all containing resources or to tag some resources individually.

If you see that something is not working as expected or become cumbersome, don’t be afraid to change it to suit your needs.

## When automation is harmful

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924034/fc931c1b-d81e-4c73-a2a6-ce98db555260.png align="center")

With the introduction of new [Azure Policy effects](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects) that allow you to modify resources during their deployment or run remediation tasks on existing resources, the temptation to automate resource configuration on the fly might be very strong. If I were you, I would think twice before creating Azure Policies that modify my resources. Let me illustrate why.

Imagine that you have a policy in your subscription that modifies resource configuration during the deployment based on some criteria, e.g., the tag’s value or its presence in the original deployment template. Now, each time you deploy a resource that violates that policy, the configuration of that resource will be changed so the resource is compliant. Sounds good, right? No really.

In real-world scenarios, when multiple people are involved in project development or service operation, such “benefits” might not be evident for all team members and cause unwanted system behavior, painful hours of debugging and confusion. You deploy configuration A, but due to some background automation, you get configuration B. All the beauty of [declarative syntax](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview#why-choose-resource-manager-templates) goes to hell.

Another example of automation doing more harm than adding value is using [Azure Policies to automatically tag resources with some default values](https://docs.microsoft.com/en-us/azure/governance/policy/samples/apply-tag-default-value). When used unintentionally, this approach might cause “tag hell” in your subscription(s), as I mentioned in the first section: lots of tags that mean nothing or have no use.

So, always remember that “with great power comes great responsibility.” If you want to create some automation based on your tags, think carefully about the side effects.

Also, [my Azure Policy repository on GitHub](https://github.com/andrewmatveychuk/azure.policy) contains sample ARM templates for Azure Policy definitions that enforce or inherit tags.