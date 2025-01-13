---
title: "Practical aspects of running a CMDB for Azure resources: Tips"
seoDescription: "Practical CMDB management in Azure: focus on design, deployment, governance, and automation for efficient operations"
datePublished: Wed Dec 02 2020 09:39:44 GMT+0000 (Coordinated Universal Time)
cuid: cm5b67beo000609l26gib7gzt
slug: practical-aspects-of-running-a-cmdb-for-azure-resources-tips
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925922/16beee76-8131-4386-9c7d-e6edc2248771.png
tags: azure, configuration-management, itil, cmdb

---

In the second part of this series, I would like to focus on the Azure-specific aspects of implementing a [CMDB](https://en.wikipedia.org/wiki/Configuration_management_database). It will be more of a collection of best practices than a prescriptive implementation guide, as there is no single right answer regarding the best CMDB option and related processes.

> To better understand the reasoning for the ideas in this post, I suggest checking out [the first part of the series](https://andrewmatveychuk.com/practical-aspects-of-running-a-cmdb-for-azure-resources-fundamentals), which introduces the basic concepts of Azure tags and why they can be your best friends when establishing your operational processes for Azure resources.

I grouped the tips around four areas: design, deployment, governance, and automation. However, that somewhat subjective grouping is used to structure the post’s information rather than to represent a standard to follow.

## Design

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922694/dbd430d6-99d9-4505-921a-545f42f31cf1.png align="center")

First of all, remember that Azure provides you with built-in logical containers, aka [resource groups](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups). There are a few essential points to understand about them:

* They are intended to group related resources that belong to the same application/service and should be managed as a group.
    
* Most of the Azure resources you deploy can be placed in a resource group only.
    
* The resource group structure is flat – you cannot build a hierarchy or contained resource groups.
    
* They are NOT folders, organizational units (OU), or databases for contained resources.
    

I know that those resource group basics might sound boring, but most cloud resource operations issues originate from misunderstanding or neglecting such simple things. For example, most attempts to organize cloud resources fail because people try to manage each resource individually. Of course, you can keep track of a hundred resource items or so, but usually, it is not the case when you really need a CMDB. In enterprise-scale environments, the counter of provisioned resources easily exceeds thousands and tens of thousands of configuration items.

> Tip # 1. Organize your CMDB processes around resource groups and not individual resources.

Suppose you start looking to assign resources in the same resource group to different owners or cost centers. In that case, you likely mix various applications/services that should be separated into different resource groups. Here some people might complain about the inability to have more sophisticated resource grouping like nested resource groups. In my opinion, the Azure product team made a very wise decision about implementing the flat structure for the groups. From my non-Azure experience, e.g., managing overlapping [GPOs](https://en.wikipedia.org/wiki/Group_Policy), excessive complexity is the worst enemy of a good design.

Next, remember that you have a wide range of tools to organize your resources from management, billing, and access perspective in Azure. Yes, I’m talking about [management groups](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview), [subscriptions](https://docs.microsoft.com/en-us/learn/modules/create-an-azure-account/4-multiple-subscriptions), and [Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis). Their proper application can elegantly solve many resource management challenges. For example, you can create a hierarchical structure of management groups reflecting a high-level organizational chart or governance model for running your workloads. You can provision multiple(!) subscriptions for billing, workload, and access segregation and organize their management with the management groups. You can create security groups and use built-in or custom Azure roles to assign inherited permissions to different scopes. Just try to keep things simple, and don’t overcomplicate your solution design.

> Tip # 2. Use proper tools for your goals.

For example, in some cases, it makes more sense to place your resources in multiple subscriptions, which are billing boundaries, rather than maintaining a complex structure of cost center tags, chargebacks, and related reports.

Lastly, be aware that you can use Azure tags at different levels – [resources, resource groups, and subscriptions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources). Technically, tags from the higher levels are not inherited per se, but you can emulate such a behavior (more on that later in the post). In other words, if all resources in a designated scope should be labeled with the same tag value, you don’t have to tag each resource individually. You can assign that tag to the parent resource group or subscription.

> Tip # 3. The more resources you can manage as a group, the less complicated your tagging convention for them will be.

Certainly, there are some [limitations for Azure tags](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources#limitations), but it is unlikely that you exceed them. If you do, you are probably doing something wrong or trying to solve your problem with improper tools.

## Resource deployment

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923656/e3b1d8d5-8e4d-4559-a01a-cbf21b7b4d7d.png align="center")

Now, let’s talk about keeping your cloud infrastructure well-organized from the resource deployment perspective.

Regardless of the specific deployment method you use – Azure CLI, ARM templates, or Azure PowerShell – all of them support Azure tags in their deployment specification, as the tags are a feature of [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#consistent-management-layer). Whether you create your resources manually through a wizard on the portal or programmatically as a part of your automated deployment pipelines, you should use [Azure tags](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources) to mark the resources with appropriate labels from the moment of their creation.

> Tip # 4. Apply tags to resources when they are created.

Don’t leave this for later. I saw too many Azure environments where people deployed lots of (cool!) cloud services without a second thought about operating and maintaining them in the future. The result was predictable: lots of cloud services were left to their own devices, and nobody was eager to do an inventory and bring everything in order.

Ideally, you should not deploy any resources without understanding how you will run them. Of course, it is not always possible to identify all tagging requirements at the early stages. Still, a seasoned cloud architect usually knows what questions to ask stakeholders and what necessary guardrails to implement. If you don’t have one in your team, request consultancy on that matter at least. As Benjamin Franklin said: “*It is easier to prevent bad habits than to break them.*”

Surely, you can lower your expectations of resource tagging in cloud development environments, provided that you run them as sandboxes with no obligations to maintainability and configured [budget limits](https://docs.microsoft.com/en-us/azure/cost-management-billing/manage/spending-limit). However, the production ones require more rigorous quality gates for resource deployment. Hopefully, there are plenty of tools for build and deployment automation, like [Azure DevOps Pipelines](https://docs.microsoft.com/ru-ru/azure/devops/pipelines/), that can ensure a consistent approach for your deployment process and ensure the required Azure tags are created as part of the process.

> Tip # 5. Deploy your cloud resources as part of your automated deployment process only.

If you need to update your tags, e.g., change the cost center for your application, you can edit them once in your deployment templates/scripts ([Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_code) practice) and let the deployment pipeline do the job.

> Tip # 6. Updating your resource tags should be performed via the same pipelines you use to deploy the resources.

Here, many Azure practitioners can object and provide many counterarguments where end-to-end deployment automation is not feasible or impractical. Indeed, it is great when you can have a fully automated process for cloud resource deployment, but it can be quite costly to implement, as with any automation. Apart from that, in large organizations with many engineering teams, the level of cloud expertise and deployment automation can vary from team to team. So, how can we be sure that all cloud resources deployed in an organization are tagged appropriately?

## Governance

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924290/afc24c4b-d930-4d61-a8ac-e94694f290ee.png align="left")

Wouldn’t it be great to have a tool to audit Azure resource configuration (including tags) and, even better, prevent misconfiguration? The good news, such a tool exists, and it is called [Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview). I’m not going into details here on how Azure Policy works as it has many applications. Instead, let’s focus on what Azure Policy can do with resource tags.

The Azure product team already provided us with a few [built-in policy definitions for tag compliance](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/tag-policies) that you can use to:

* require a tag (and its value) on resources;
    
* add or replace tags;
    
* inherit or append tags from the resource group or subscription.
    

For instance, by applying the ‘require a tag’ policy to a specific scope like a subscription, you prevent creating resources without that tag. In other words, if somebody forgets to specify that tag in a deployment (by any technical means), that deployment will fail, and no untagged resources will be provisioned.

> Tip # 7. Use Azure Policy for tag compliance.

However, from a practical point of view, demanding tags in each resource definition might be impractical — remember Tip # 3. If you design your Azure workloads to have one application/service per resource group, it will mostly be enough to enforce tag compliance at the resource group level, as there are usually far fewer groups than resources. If you need some of those tags to be present at the resource level, use the corresponding policy to inherit them so that deployed resources will be enriched with those tags.

> Tip # 8. Enforce tags at the resource group level, inherit the tags on contained resources if needed.

You don’t always start from scratch in real life and sometimes have to deal with Azure environments lacking appropriate tags for a CMDB and related operational processes. In that case, you can use Azure Policy to audit tag compliance before putting the tag enforcement in place. At the time of writing this, Azure doesn’t have built-in policies for auditing tags. Still, they are relatively easy to implement – you can use [sample definitions for custom policies in my Azure Policy repository on GitHub](https://github.com/andrewmatveychuk/azure.policy/tree/master/other-samples/policies/definitions/tagging) for that.

> Tip # 10. Audit your tag compliance before restricting untagged resources.

## Automation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925330/ddc678f4-711f-4437-b316-c0ec197d581e.png align="center")

Lastly, remember that cloud services are about flexibility and change. Although concepts such as [immutable infrastructure](https://www.digitalocean.com/community/tutorials/what-is-immutable-infrastructure) and disposable environments exist, cloud resource management is not limited to deployments only.

Imagine a person owning an application in your organization decided to leave the company, or an application needs to be charged to a different cost center for whatever reason. Those practical cases have little to do with deployment or compliance. Nevertheless, they are far more important for CMDB maintenance than you might think.

As I mentioned in [the previous part](https://andrewmatveychuk.com/practical-aspects-of-running-a-cmdb-for-azure-resources-fundamentals), Azure tags are just text labels and not connections to the configuration items they represent (user accounts, cost centers, business applications, etc.). In essence, you should have a change management process for each tag you use. Such a process should explicitly define events that trigger its execution, the procedure to follow, and the desired outcome. From my experience, a one-page diagram should be enough for the process description. If you need to add a long explanation to the chart to make it understandable, you had better review and simplify the process.

> Tip # 11. Have a change management process for each Azure tag you use.

Obviously, manually updating hundreds of tags or running an inventory for thousands of resources is far from efficiency ideals. Assuming you have those change management processes (diagrams) in hand, you should look for ways to automate them. There are plenty of them.

For example, if you manage your Azure resource via code (IaC), you can implement tag updating via deployment pipelines, as I mentioned previously. If that is not the case, you can look into implementing an Azure Automation runbook or a Logic App to be triggered from a third-party HR, finance, or planning system. The key point is that without automation, it is almost impossible to ensure tag consistency on a large scale.

> Tip # 12. Automate tag updates.

When designing your automation for tag updates, remember that Azure tags are just a means to implement your CMDB maintenance processes. So, think of a process you would like to implement first and the technical details second. From my experience, simplifying a cumbersome process will benefit you much more than any sophisticated technology, but that shall be a topic for another blog post.