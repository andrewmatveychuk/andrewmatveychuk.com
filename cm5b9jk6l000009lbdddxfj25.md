---
title: "How to authenticate to Azure with managed identities from non-Azure servers"
seoDescription: "Learn to authenticate Azure resources from non-Azure servers using managed identities, enhancing security without storing credentials"
datePublished: Tue Jul 02 2024 11:00:17 GMT+0000 (Coordinated Universal Time)
cuid: cm5b9jk6l000009lbdddxfj25
slug: how-to-authenticate-to-azure-with-managed-identities-from-non-azure-servers
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736867463233/bd3996da-18d8-4ec5-9dbe-75c113387747.png
tags: authentication, azure, security, entra-id

---

In the third post in my series about secure authentication to Azure services, we will explore how to access Azure resources from servers hosted on-premises or in other clouds without storing any credentials, like client secrets or certificates, on them.

For the previous posts in this series, please check the following articles:

* [How to securely authenticate your applications to Azure services](https://andrewmatveychuk.com/how-to-securely-authenticate-your-applications-to-azure-services/)
    
* [How to use certificate credentials to authenticate to Azure services](https://andrewmatveychuk.com/how-to-use-certificate-credentials-to-authenticate-to-azure-services/)
    

## A secure password is the one you don’t know

As I mentioned in my first post, I encourage people to [use managed identities to authenticate to Azure services whenever possible](https://andrewmatveychuk.com/how-to-securely-authenticate-your-applications-to-azure-services/). Using managed identities greatly simplifies the management of communication credentials in your application. Plus, it helps you mitigate many security risks related to storing and using the app’s secrets, passwords, keys, etc. You no longer need to worry about leaked passwords, rotating your credentials, or ensuring that different environments use different credentials.

When hosting your applications in Azure, managed identities should be your number one choice for most authentication scenarios between application components. However, what do you do in more common hybrid or multi-cloud setups when your application infrastructure is partially outside Azure?

In that case, you can extend Microsoft Entra ID’s identity functionality beyond Azure using [Azure Arc](https://learn.microsoft.com/en-us/azure/azure-arc/overview) for free.

> Azure Arc provides many more useful features other than utilizing managed identities, but they are not the focus of this article.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922673/ce68034e-d996-4d29-aaff-2a7bff9fe3fd.png align="center")

Technically, after you install [the Azure Connected Machine agent](https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview) on your non-Azure server, it will [link the server with the connected machine’s Azure identity](https://learn.microsoft.com/en-us/azure/azure-arc/servers/managed-identity-authentication?ref=andrewmatveychuk.com#security-overview) and create a local identity endpoint that can be used to request an access token for your application.

## How to use managed identity on Azure Arc-enabled servers

As I explained in my post about [using certificate credentials to authenticate to Azure services](https://andrewmatveychuk.com/how-to-use-certificate-credentials-to-authenticate-to-azure-services/), you can configure your application to use specific identity types in a few ways.

The first and most preferable option is to rely on [the built-in fallback mechanism](https://learn.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme?view=azure-dotnet&ref=andrewmatveychuk.com#defaultazurecredential) of the [DefaultAzureCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential) class. If you don’t set [the environment variables](https://learn.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme#environment-variables), it will try to authenticate with a managed identity as the third option in the sequence. In practice, that means you don’t need to change anything in your code, and the same code that worked with authentication using client secret or client certificate will continue to work provided that you configured the required permissions for the managed identity to access the target Azure resource 👇:

%[https://gist.github.com/andrewmatveychuk/45e9c50b0be362cb90a75de0c3262868] 

> **Note**. [The account used to run your application must be a member of a specific group](https://learn.microsoft.com/en-us/azure/azure-arc/servers/managed-identity-authentication?ref=andrewmatveychuk.com#prerequisites) on an Azure Arc-enabled server to access the identity endpoint and get access tokens. Otherwise, you might get an error like the following:  
> Authentication Failed. ManagedIdentityCredential authentication failed: Access to the path 'C:\\ProgramData\\AzureConnectedMachineAgent\\Tokens\\292daa9f-1794-43f4-a246-6f6cc6ca4e03.key' is denied.  
> Also, if you run your application interactively, you need to do it with elevated permissions as administrator.

The second option is to use application settings like [configuration in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration) to explicitly tell your application to use a managed identity to connect to an Azure service. For example, the code below is the same that I used to connect from the sample .NET Worker service using certificate credentials configured in an appsettings.json file 👇:

%[https://gist.github.com/andrewmatveychuk/058213eac0d8ac301b8c29fde8404c19] 

Now, you can [tell the app to specifically use the managed identity](https://learn.microsoft.com/en-us/dotnet/azure/sdk/authentication/create-token-credentials-from-configuration#create-a-managedidentitycredential-type) without changing your application code:

%[https://gist.github.com/andrewmatveychuk/5949b3497182f9a5aa466998b88e8cf7] 

As you can see, compared to [using certificate-based authentication to Azure services](https://andrewmatveychuk.com/how-to-use-certificate-credentials-to-authenticate-to-azure-services/), you no longer need to worry about configuring, securing and rotating connection credentials with managed identities. All of it is done for you. You can focus more on your application functionality rather than on non-customer-facing tasks. Moreover, you can easily switch between different authentication methods by following the recommended approaches to using Azure Identity libraries in your application.

> You can also check the sample code in my GitHub repository: [azure.authentication-samples](https://github.com/andrewmatveychuk/azure.authentication-samples?ref=andrewmatveychuk.com).

Have you tried to use managed identities on Azure Arc-enabled servers? What was your experience with it? Share your thoughts in the comments below 👇