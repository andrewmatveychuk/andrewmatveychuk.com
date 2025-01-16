---
title: "How to access private PowerShell repository from Azure pipeline"
datePublished: Thu Apr 09 2020 04:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5pe87pf000w09ld3fqwg994
slug: how-to-access-private-powershell-repository-from-azure-pipeline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737041748344/262c439c-efca-4c25-a2b1-ec6f621e5c9f.png
tags: powershell, cicd, azure-devops

---

If your PowerShell repository is hosted on a public Azure Artifacts, working with that repository is no different from regular operations with [PowerShell Gallery](https://www.powershellgallery.com/), for example. However, many enterprises prefer to keep their code base and build artifacts private for multiple reasons: legal, security, intellectual property, internal policies, and others. In this case, you might choose to go with a private Artifacts feed, and here the challenge begins.

## Hosting private PowerShell repository on Azure Artifacts

Thanks to [the community effort](https://medium.com/@jsrice7391/using-vsts-for-your-companys-private-powershell-library-e333b15d58c8), Microsoft has already published a detailed guide on using [Azure Artifacts as a private PowerShell repository](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library). Also, if you examine the cmdlets from the [PowerShellGet](https://docs.microsoft.com/en-us/powershell/module/powershellget/) module, you will notice that many of them accept a [PSCredential](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential) object as a means of authentication. You can create a PSCredential object from your personal access token (PAT) and [use that object as an input parameter for registering a private repository, look for modules in it and install them](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library#connecting-to-the-feed-as-a-powershell-repo).

## Accessing private PowerShell repository from Azure Pipelines

The documented approach with credentials works just fine when working with a private PowerShell repository hosted on Azure Artifacts **from the console**. Basically, the command logic is quite simple:

%[https://gist.github.com/andrewmatveychuk/5a4ed8b58bda02e55304d299d14d2c92] 

As you might see, the only difference from working with public PowerShell repositories is using the optional Credential parameter. You create your credential using your email and the generated personal access token and pass it to cmdlets.

However, if you try to run the same script **in the pipeline context**, you might encounter the following error:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681919653/7e095548-5210-4ce0-8c83-83516f740ef3.png align="center")

> \[Minimal\] \[CredentialProvider\]DeviceFlow: &lt;YOUR REPOSITORY\_FEED\_URL&gt;  
> \[Minimal\] \[CredentialProvider\]ATTENTION: User interaction required.  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code &lt;AUTH\_CODE&gt; to authenticate.  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> \[Error\] \[CredentialProvider\]Device flow authentication failed. User was presented with device flow, but didn't react within 90 seconds.

When running in the pipeline, PowerShell switches to the device flow to authenticate. To be honest, I don’t know the exact reason for such behavior as of now, but I can only assume that it might be related to the way the [NuGet authentication plugin works](https://docs.microsoft.com/en-us/nuget/reference/extensibility/nuget-cross-platform-authentication-plugin).

Strangely, after a few unsuccessful attempts, the cmdlet manages to complete authentication and register the repository. However, this still negatively impacts the build process by increasing the overall pipeline run time.

A workaround for that issue is switching from the [Register-PSRepository](https://docs.microsoft.com/en-us/powershell/module/powershellget/register-psrepository) to a more versatile [Register-PackageSource](https://docs.microsoft.com/en-us/powershell/module/packagemanagement/register-packagesource). Apart from that, you can use the ‘[System.AccessToken](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml#systemaccesstoken)’ variable to authenticate a pipeline to a private Azure Artifacts feed so that you do not worry about PAT expiration. For example, to register a private repository:

%[https://gist.github.com/andrewmatveychuk/54bedbf5e891a8318a7f2eb765ddab16] 

After implementing that workaround, the PowerShell cmdlets execute without any exceptions or errors. You can register your private PowerShell repository on a build agent and install the module your build process depends on.