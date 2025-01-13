---
title: "How to deploy linked ARM templates from private Azure DevOps repositories"
seoDescription: "Learn how to deploy linked ARM templates from private Azure DevOps repos using a consistent, automated solution with Azure DevOps pipelines"
datePublished: Thu Dec 19 2019 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5moemt5000509lad9q5dmz2
slug: how-to-deploy-linked-arm-templates-from-private-azure-devops-repositories
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922775/5dfb342d-9fbf-4992-b48e-e1d926985063.png
tags: azure, cicd, azure-devops, arm-templates

---

If you are reading this, you probably already know that Azure Resource Manager should be able to access linked ARM templates via the [provided URI](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-linked-templates#linked-template) while deploying the master template. You cannot use local files or URI locations that require some form of password-based authentication or HTTP authentication headers.

A Microsoft-suggested approach to working with linked templates is [to copy them to a storage account](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-tutorial-create-linked-templates#upload-the-linked-template) and [make them externally accessible](https://docs.microsoft.com/en-us/azure/azure-resource-manager/secure-template-with-sas-token?tabs=azure-powershell#provide-sas-token-during-deployment) with SAS tokens. Sounds simple enough! However, the official samples are somewhat inconsistent from my point of view: some parts are performed in ARM templates, some require PowerShell scripting, and the life cycle of the storage account for linked templates is not addressed at all.

So, how do we create a deployment solution for “private” linked ARM templates that is simple, consistent, easily repeatable and fully automated with Azure DevOps pipelines?

## Prerequisites

You will need four primary building blocks for your solution:

* main or “master” ARM template for deploying the services you need from corresponding linked templates;
    
* linked ARM templates for your specific Azure services or sub-parts of your infrastructure;
    
* an ARM template to create a storage account for deployment artifacts, i.e., linked templates and other needed files;
    
* an ARM template for the resource group you are going to deploy into.
    

That’s all. No PowerShell is required unless you must perform some pre- or post-deployment tasks that might be relevant in your specific case.

## Deployment order

The whole deployment process will be performed according to the following order:

1. Create an empty resource group for your deployment.
    
2. Create a storage account in that resource group to store deployment artifacts.
    
3. Copy linked templates and any other needed deployment artifacts to a container(s) in that storage account. Get the storage account URI and SAS token.
    
4. Deploy your main ARM template that references linked templates in the storage account.
    

The sample Azure DevOps pipeline to implement such deployment might look like the following:

%[https://gist.github.com/andrewmatveychuk/054a1146311507b53c64bbfbf7bfa166] 

Let’s look at the pipeline flow in detail.

First of all, pay attention to the job type – it is a good practice to run your tasks that perform actual deployment inside a [deployment job](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs). With this job type, you can implement different deployment strategies, target [the environment for deployment](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/environments), create gated deployments that require manual approval, etc.

Secondly, to pass the outputs of ARM deployments in the pipeline, you should specify an environment variable in [Azure Resource Manager (ARM) Template Deployment tasks](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks/AzureResourceManagerTemplateDeploymentV3) and use simple PowerShell to parse those outputs and create pipeline variables you can reference in the following steps. This might be very handy if you dynamically create disposable environments for testing or implement a versioned [immutable infrastructure](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure).

Thirdly, the [Azure File Copy task](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks/AzureFileCopyV3) provides options to create pipeline variables for storage account URI and SAS token. Plus, you can set an expiration time for the SAS to generate, so it is relatively short-lived and valid only for the time needed to run your deployment.

Lastly, you can use the ‘overrideParameters’ option in the Azure Resource Manager (ARM) Template Deployment task to override the deployment parameters for the storage account URI and SAS token on the fly.

The steps to test and validate ARM templates are not included in the example for the sake of simplicity. As a rule of thumb, remember to extend this starter pipeline with additional stages to perform automated testing.

Optionally, you can also add tasks to delete deployment artifacts from the storage account if performing incremental deployments or a task to remove the storage account itself.

## Template linking

To repeat the described approach for deploying linked ARM templates, you [should design your main templates to take the storage account URI and SAS token as input parameters](https://github.com/starkfell/100DaysOfIaC/blob/master/articles/day.42.deploy.nested.arm.templates.using.storage.accounts.in.yaml.pipeline.md#1-preparing-the-arm-templates). Besides, to make the templates more readable, you can construct the URIs for linked templates based on those parameters in the variables section:

%[https://gist.github.com/andrewmatveychuk/5b03f87fa3784fcaf1f76a57fd5b2f5b] 

Implementing this approach as a standard for your main templates will help you repeat this deployment pattern confidently and spend less time debugging referencing errors.

## Boilerplate

You can use the approach described in your Azure DevOps pipelines or use the [sample repository on GitHub](https://github.com/andrewmatveychuk/azure.linked-arm-templates) as a draft solution for your deployment stages.

P.S. Depending on your requirements, you might decide to keep the storage account for deployment artifacts outside the deployment resource group, adding unnecessary complexity to your deployment process. If I were you, I would use a single resource group as a logical boundary for your deployments whenever possible.