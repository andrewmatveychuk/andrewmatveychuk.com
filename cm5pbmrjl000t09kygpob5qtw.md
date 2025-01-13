---
title: "How to deploy Azure Policy from an Azure DevOps pipeline"
seoDescription: "Learn how to deploy Azure Policy from an Azure DevOps pipeline with ARM templates and configure necessary permissions"
datePublished: Thu Jan 23 2020 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5pbmrjl000t09kygpob5qtw
slug: how-to-deploy-azure-policy-from-an-azure-devops-pipeline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921701/6fc55233-2d7b-49e2-835b-bc5561f38014.png
tags: azure, cicd, azure-devops, azure-policy

---

After publishing my posts about deploying [Azure Policy](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates) and [Azure Policy initiatives](https://andrewmatveychuk.com/using-arm-templates-to-deploy-azure-policy-initiatives) from ARM templates, I got a few questions about performing such deployments from Azure DevOps pipelines. Indeed, there are a few things to pay attention to. So, I will try to shed some light on them in this article.

## Azure pipeline tasks

If you define your policies in ARM templates as [I do](https://andrewmatveychuk.com/how-to-deploy-azure-policies-with-arm-templates), you can include a step with the [Azure Resource Manager (ARM) Template Deployment Task](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks/AzureResourceManagerTemplateDeploymentV3), which is the successor of the older [Azure Resource Group Deployment Task](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks/AzureResourceGroupDeploymentV2), in your pipeline. This new version has many more [deployment options](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks/AzureResourceManagerTemplateDeploymentV3#parameters-of-the-task), especially targeting your deployments to a resource group, subscription or management group. It also supports [Validate mode](https://docs.microsoft.com/en-us/rest/api/resources/deployments?redirectedfrom=MSDN#Deployments_Validate) for your deployment, so you can check first whether your template is syntactically correct. Additionally, if you target your deployments to multiple environments requiring different configurations, you can override template parameters and keep your repository clean from multiple parameter files.

The Azure DevOps product team recently updated the pipeline UI to support that new ARM template deployment task, but I suggest always checking the [Azure Pipelines Tasks repository on GitHub](https://github.com/microsoft/azure-pipelines-tasks) first for recent changes and updates.

If you have some build scripts in your project to compile and deploy Azure Policy artifacts from them in batch, then you can use the well-known ‘[New-AzDeployment](https://docs.microsoft.com/en-us/powershell/module/az.resources/new-azdeployment)’ and ‘[Test-AzDeployment](https://docs.microsoft.com/en-us/powershell/module/az.resources/test-azdeployment)’ PowerShell cmdlets for subscription-level deployments, which is usually the case for Azure Policies. [Azure PowerShell tasks](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks/AzurePowerShellV5) are at your service to run the scripts.

Pay attention to the fact that both cmdlets support input of ARM template parameters from a ‘TemplateParameterObject’, which is a simple hashtable. That option allows you to implement a similar override pattern for template parameters and perform conditional deployments.

For example, the following code reads a template parameter file for policy assignment, extracts the parameters into a hashtable and changes or adds additional deployment parameters based on input script parameters or its variables:

%[https://gist.github.com/andrewmatveychuk/4f2fb481ba27a458da054b7e8dedca4c] 

Apart from that, you can create a [master (or main) template for deploying your custom policy and initiative definitions and their assignment from linked templates](https://github.com/andrewmatveychuk/azure.policy/tree/master/main-template). If you are working in a private repository, you might consider [creating a pipeline that copies your linked templates to a storage account during deployment](https://andrewmatveychuk.com/how-to-deploy-linked-arm-templates-from-private-azure-devops-repositories/).

As you can see, there are plenty of options for deploying Azure Policy from a pipeline, but the final choice depends on your objectives, limitations and used build/test/deploy patterns.

## Configuring permissions to deploy Azure Policy

When you try to run your pipeline that has deployment tasks for Azure Policy artifacts for the first time, you most likely will get the following error:

> Authorization failed for template resource '&lt;your\_policy\_name&gt;’ of type 'Microsoft.Authorization/policyDefinitions'. The client '’ with object id '' does not have permission to perform action 'Microsoft.Authorization/policyDefinitions/write' at scope '/subscriptions/046a3b8b-ca72-46b7-8bd6-4a7cc5357741/providers/Microsoft.Authorization/policyDefinitions/&lt;your\_policy\_name&gt;’

That error message indicates some permission issues, as some processes cannot perform the ‘**write**’ operation in the ‘Microsoft.Authorization/policyDefinitions’ namespace. So, let’s sort it out piece by piece.

Firstly, the authorization has failed for a specific resource in your ARM template, and that resource type is Azure Policy. Secondly, the authorization did not succeed for a client with a particular ID, which is represented as GUID, when it tried to create a new resource in the target namespace and target scope. At the same time, if you try deploying the same ARM template manually, the deployment completes successfully provided that you are an administrator on the subscription and the template is valid.

As the deployment tasks run in the pipeline context, you might assume that there is something wrong with the permissions for the service connection to your Azure subscription, which is used in your pipeline. So, let’s check what configuration we have for that service connection in your Azure DevOps project:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681919547/dcd67e76-ed7b-499d-b972-9ccf2ce8d860.png align="center")

To get the extended information on the screen above, click on ‘**use the full version of the service connection dialog**’ at the bottom of the default connection properties windows.

Here, you might notice that the service principal client ID and the GUID from the error message are the same, so our assumption about the pipeline context was right.

Next, let’s check Access control for your subscription on the Azure portal. You will see that there are a few role assignments that are named similar to your Azure DevOps projects:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681920403/41981816-74f0-4b3a-98bc-7ea675210497.png align="center")

When looking into the assignment properties related to your project, you also might notice the same ID:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681921042/0d35e98c-b036-4135-b85a-f2b60d441189.png align="center")

By default, [Azure DevOps grants ‘Contributor’ permissions](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/azure-rm-endpoint?view=azure-devops#what-happens-when-you-create-a-resource-manager-service-connection) for the service principals used to authenticate pipelines to Azure, which are fine for most regular deployments. However, this built-in role [doesn’t have permission to deal with Azure Policy resources](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor).

Granting our service principal extensive administrative permissions in the subscription is not considered a good security practice, so there should probably be another option. Indeed, the built-in ‘[Resource Policy Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#resource-policy-contributor)’ role exists, which has the permissions we need to work with Azure Policy.

You can use the following PowerShell command to assign that additional role to your service principal and re-run the pipeline:

`New-AzRoleAssignment -ObjectId (Get-AzADServicePrincipal -ApplicationId <your_service_principal_client_ID>).Id -RoleDefinitionName 'Resource Policy Contributor'`

Now, you shouldn’t see the permission error anymore.

I hope this information will help you to create an Azure DevOps pipeline for Azure Policy and deploy policies and policy initiatives as part of your deployment process.

If you have any questions or suggestions, go ahead and post them in the comments!