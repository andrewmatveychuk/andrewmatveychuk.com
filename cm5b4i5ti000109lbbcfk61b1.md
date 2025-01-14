---
title: "Azure Policy Best Practices"
datePublished: Tue Dec 13 2022 06:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b4i5ti000109lbbcfk61b1
slug: azure-policy-best-practices
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736871007237/0f4de9cc-533b-4d07-9dc4-98f707807444.png
tags: azure, azure-policy

---

Mastering a new tool might be challenging, so here I will share my best practices for working with Azure Policy. Those tips are based on my experience and intended to complement my [Azure Policy Starter Guide](https://andrewmatveychuk.com/azure-policy-starter-guide). Although most of them will be about pretty simple things, to my mind, it’s usually the basics that people tend to overlook and suffer the consequences of their neglect. So, let’s start.

## Use parameters

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922612/0f778bb2-ef06-4549-bad1-ca435bfdfd6c.png align="center")

It might seem obvious that to make your code more reusable, you should use input parameters to control its behavior. Still, you can come across many examples, even with the built-in policies, when their authors didn’t bother with parameters and hardcoded some property values in the policy code. In such cases, what could have been accomplished with a simple update of policy assignment parameters now requires updating your Azure Policy definition, probably removing the current policy assignments and the definition if it’s incompatible with the new one, deploying the new policy definition into your target scope, and creating new assignments for it. Sounds like a ton of work, right? Now, imagine you need to update a dozen of those non-parametrized policies. On a second or third occasion, it’s not fun at all, and you probably start thinking about making your policy definition more adjustable to changing business requirements.

Let me illustrate how parametrizing a single policy value can instantly make your custom Azure Policy twice as valuable. If you have worked with Azure Policy before, you might know that they can have different [effects](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effects). The **Audit** effect is the most used and usually the simplest to implement. You might find many custom policy definitions where it’s defined as follows:

%[https://gist.github.com/andrewmatveychuk/1770e42e4f80104934bc326e9a79aa56] 

That’s what [the official documentation](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effects#audit-example) says.

However, if you go the extra mile and check a few Azure Policy definitions implementing the same effect, you might notice that the policy developers tend to define the policy effect as an input policy parameter like that:

%[https://gist.github.com/andrewmatveychuk/02fc8cedf53e98730b02da8e98853f7b] 

Now, a simple switch flip can change your custom policy behavior from simply auditing for non-compliant resources to preventing them from deploying. It’s the same policy rule and the same logic, but twice as helpful thanks to that simple change. Now, when a business tells you they are tired of chasing Azure resource owners or fixing non-compliant configurations and want to block any such deployments from happening, you can implement such a control in the blink of an eye.

## Please keep it simple

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923588/57e3809f-69be-40d0-ba75-1307159680b0.png align="center")

Many [ARM template functions can be used in policy rules](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#policy-functions) to implement some advanced scenarios. You can reference resource properties, evaluate arrays, define conditional logic, work with strings, and do many other things. Sample Azure Policies for Guest Configuration or Kubernetes clusters can change your view of how powerful and complex(!) that tool can be. However, the complexity of your policy code doesn’t mean it’s a good thing to do all the time.

For instance, I occasionally read my own Azure Policy code from six or so months ago and have difficulty understanding how particular ‘fancy’ things work there. I often use my work notes and even read my old blog posts to recall why I liked them or how specific code works. Now, look at this from other people’s perspectives. Will it be easy for them to maintain and modify your custom policies in the future?

In my experience, 80% of using Azure Policy is all about simple stuff like auditing, modifying tags or creating guardrails for your cloud environments in Azure. So, why not keep its implementation simple, too? If it’s hard to do with Azure Policy, then it’s probably not the right tool for your task. Look at other Azure services, many of which have integrations with the policy APIs.

Another extreme, which is opposite to not using input parameters in Azure Policy, is defining parameters for everything and trying to make a Swiss knife out of your policy. Of course, you can specify the default values for those parameters so that you don’t have to provide them whenever creating a policy assignment. Still, it makes your policy definition harder to read and understand. For example, if your policy logic is about Azure VMs, there is no need to supply that resource type as an input parameter. Or, if you want to [audit Azure Hybrid Benefit usage](https://andrewmatveychuk.com/audit-and-enable-azure-hybrid-benefit-using-azure-policy) across different eligible resource types, it might be better to define those controls in separate per-type policy definitions and combine their assignment with a policy initiative. That will make your solution cleaner and easier for other people to understand.

## Test for side effects

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924237/b2d3758c-9841-4d6b-b31d-cf8e1ddd30cd.png align="center")

I might sound repetitive in my posts about Azure Policy, but please do test the policies first before assigning them to a production scope. It’s crucial in the case of the policies that modify, deploy or deny resources. Because most Azure Policy assignments happen at the subscription or management group levels, [the blast radius](https://en.wikipedia.org/wiki/Blast_radius) of such changes is usually huge. Also, cloud adoption at organizations tends to evolve from prototyping with no or few rules first to catching up with cloud governance later. So, by the time you start putting your policies in place, hundreds or even thousands of existing resources might not comply with them.

Let’s take, for example, [the built-in policy that defines the list of allowed resource locations](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/AllowedLocations_Deny.json). It’s commonly mentioned in cloud governance practices that suggest limiting the deployment of your Azure services to specific regions. There might be multiple reasons for that, but my point here is not about cloud governance. At first glance, it seems easy to assign that policy and be done with that control. However, if you have existing resources that don’t comply with that policy, you will effectively block them from further modification. That’s because, from the Azure Resource Manager API perspective, the same ‘write’ endpoints are used for creating and updating(!) existing resources. Now, we have a problem here.

In the case of community-crafted Azure Policies, you should be even more watchful. The fact that somebody already created a policy that seems to fit your needs ideally doesn’t mean that you should go and assign it straight ahead. It’s no different from copy-pasting a code from Stack Overflow without first understanding how it works. Many such policies might have errors in their logic or be not up to date with recent Azure API changes.

So, one more time — always test your policy in a lab environment first. Even if you’ve worked with it before and know it inside out, there are likely to be edge cases that you haven’t seen yet, as every new environment might be slightly different.

## Do use custom non-compliance messages

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924944/b223484b-2831-4a40-b7fc-e3f5a7b0a67f.png align="center")

In the past, if some Azure Policy blocked an Azure resource deployment, you would get a generic error message and reference to the blocking policy at best. To say that it was frustrating is to say nothing. You need to look into deployment and activity logs to determine what that error is about and what you must fix to pass the policy checks. Developers doing their deployments from a console or an automated pipeline are usually required to go to engineers managing cloud infrastructure to troubleshoot such issues, as the console output was even less verbose.

That user experience improved significantly with the introduction of [custom non-compliant messages for policy assignments](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/assignment-structure#non-compliance-messages) and more verbose console output. Now, instead of a generic message that some policy blocked a resource deployment, you can provide a user with more helpful information like instructions on what exactly was wrong and how to fix that or a link to a knowledge base article with a detailed policy description and troubleshooting steps.

The functionality of custom non-compliance messages proved to be extremely helpful, especially for Azure Policies with the **Deny** effect. Please don’t skip out on it. Moreover, I suggest putting double effort into crafting your custom messages so they provide other users with clear instructions on how to pass the policy validation. Another person coming to you with a question (or a support request) about a policy-blocked deployment should encourage you to revisit the corresponding non-compliance message until it has enough details for self-service.

## Set up a CI/CD pipeline(s) for Azure Policy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925575/cc66c991-dea8-4cc9-9141-5e37eacacbce.png align="center")

Although Azure Policy doesn’t introduce a separate programming language in a broad sense, they are still defined in code. I’ve already authored a couple of articles on that, so feel free to check them:

* [How to deploy Azure Policy with Bicep](https://andrewmatveychuk.com/how-to-deploy-azure-policy-with-bicep)
    
* [How to deploy Azure Policies with ARM templates](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates)
    
* [Using ARM templates to deploy Azure Policy initiatives](https://andrewmatveychuk.com/using-arm-templates-to-deploy-azure-policy-initiatives)
    

As with any code, all DevOps-proven techniques, such as code versioning, automated testing, continuous integration, and continuous deployment, also apply to developing and maintaining your custom policy definitions and assignments. All the tooling for that is here and free to use. For instance, you can check my blog post on how to automate your policy deployment:

* [How to deploy Azure Policy from an Azure DevOps pipeline](https://andrewmatveychuk.com/how-to-deploy-azure-policy-from-an-azure-devops-pipeline)
    

Investing your time in setting up your automated deployment process for Azure Policy will drastically improve your development speed and increase your confidence in its quality. Even if you have no intention to develop your custom policies and plan to use the built-in ones only, it’s still reasonable to embed the creation of policy assignments in your CI/CD pipeline and use staging environments, quality checks, and gated approvals before deploying in production.

Of course, those tips are just a tiny fraction of the best practices to follow when working with Azure Policy, and I plan to update that list with other relevant highlights from my experience. If you think that I missed something important here, please feel free to comment and add your suggestion in the comment box below 👇