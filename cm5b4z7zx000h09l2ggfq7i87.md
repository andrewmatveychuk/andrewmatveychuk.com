---
title: "How to securely authenticate your applications to Azure services"
seoDescription: "Authenticate applications to Azure with managed identities, certificate-based authentication, and AWS Secrets Manager"
datePublished: Tue May 28 2024 13:00:18 GMT+0000 (Coordinated Universal Time)
cuid: cm5b4z7zx000h09l2ggfq7i87
slug: how-to-securely-authenticate-your-applications-to-azure-services
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736867522289/2bfcf0f9-75b1-4eec-bc8a-46f9476e11d1.png
tags: azure, security

---

There are multiple ways to authenticate your applications when accessing Azure services, and to be honest, authentication on its own is a vast and complex area. However, depending on your context, requirements, and application location ([Azure-hosted or hosted outside of Azure](https://learn.microsoft.com/en-us/dotnet/azure/sdk/authentication/?tabs=command-line#recommended-app-authentication-approach)), your authentication options will usually be limited to only a subset of that variety. So, instead of reviewing and understanding all possible authentication approaches in detail, it might be more practical to explain possible authentication options using a few case studies.

My primary intent with this article is to show how to eliminate using plain text credentials in application configuration options and connection strings. In the follow-up posts, I will share some code samples and configuration details to help you get started.

> **Disclaimer.** The topic of authentication and authorization in Azure is much more complex than the cases described in this article. The cases I present here don’t cover all possible authentication scenarios and should be treated as examples to help you understand the topic.

## Case 1. Use managed identity whenever possible

Imagine you have an Azure App Service, Azure Function, or Azure VM that needs to connect to some database like Azure SQL Database. The application somehow needs to authenticate to that data source. What you can usually see:

* People create an SQL user because it’s easy, or they already did it when developing locally on their machine, or they followed tutorials showcasing how to connect to a database using username and password, or for any other reason.
    
* In the best case, that user has limited access to the target database only with required permissions. In the worst case, which is more common, that user has full access to the database (or all databases) because, you know, developers don’t want to spend their time troubleshooting permission issues.
    
* The password complexity of that user is mostly far from what is considered to be a strong password. If you think of something like “Secret123”, you are close to guessing it.
    
* That username and password are used in plain text in connection strings, configuration files, or environment variables, exposing them to anybody who can access the app environment.
    
* Those credentials are shared with other applications that need access to the same database, making updating them painful and error-prone. So, forget about password rotation.
    
* To make matters worse, the same username and password are used in both development and production environments.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922822/71b8708f-1dec-4eef-9a91-7666fd5c2549.png align="center")

It looks like a terrifying nightmare for a security-aware person, and it is a harsh reality of what can be encountered even in top business-critical systems.

Besides the risk of such a user (aka a service account) being easily compromised, timely detection of its malicious use is usually not the case. In my experience, if people go so far as to make their application authentication insecure, such organizations also have little to no capabilities for detecting compromised accounts. Otherwise, they would pay more attention to that risk.

Luckily for us, the application authentication to the database in the described case can be improved with little effort. If you haven’t heard of the concept of [managed identities for Azure resources](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview), I strongly recommend familiarizing yourself with it. Their use can greatly improve your application security and boost your development experience.

A modernized version of our sample application can look like the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923770/e2c9664a-289d-48db-9c60-83032613768d.png align="center")

Simply put, your application can [utilize a managed identity provided by the underlying Azure resource](https://learn.microsoft.com/en-us/azure/app-service/overview-managed-identity) on which it’s hosted. If you have ever configured a connection from an IIS-hosted application to an on-premises SQL Server, it is similar to [using machine accounts to authenticate to target servers](https://learn.microsoft.com/en-us/iis/manage/configuring-security/application-pool-identities).

> Back then, many administrators expressed their concerns that granting access via a machine account could expose that identity to any application running on that machine. However, cloud resources, especially serverless ones, are much more granular regarding the number of hosted applications. Plus, using a managed identity is much more secure than putting usernames and passwords in plain text in files or environment variables where they can be extracted from and then used elsewhere to connect to target resources.

[The list of Azure services supporting managed identity](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/managed-identities-status) and role-based control is constantly increasing. So, it’s always worth checking if your application’s Azure resources can leverage that authentication approach.

By upgrading your application authentication from password-based to using managed identities, you:

* No longer have a password that can be compromised or shared inappropriately.  
    Don’t need to think about configuring credentials in your application.
    
* Can forget about credential rotation, as it’s done for you.
    
* Don’t have to share the same credentials with other applications, as it’s easier to configure access for another application using its managed identity,
    
* Don’t need to worry about updating your connection credentials when pushing new application versions between environments.
    

Of course, using managed identities doesn’t eliminate all security risks. You still need to grant permissions to them following the principle of least privilege to avoid excessive access. Plus, you should always consider other security risks as well. For example, if your application is vulnerable to SQL injection attacks, those attacks can be executed regardless of whether you use managed identities or not to connect to your database.

## Case 2. Prefer certificate-based authentication over password-based

You might say that everything I described in the previous case looks good if all your application parts run in Azure. However, what to do in hybrid scenarios when connecting to some Azure service from on-premises? Let’s look at what we can do in that case.

The preferred way, suggested by Microsoft, is to [register your application in an Azure tenant](https://learn.microsoft.com/en-us/dotnet/azure/sdk/authentication/on-premises-apps), creating a logical representation of it, which can be used to assign permissions and access other Azure-hosted and Microsoft-provided services. Think of it as creating an identity for your application that is used to retrieve access tokens to various services.

> Why [that authentication approach is preferable](https://learn.microsoft.com/en-us/dotnet/azure/sdk/authentication/?tabs=command-line#advantages-of-token-based-authentication) to configuring access at the destination application level? For example, you can create an SQL-contained user in an Azure SQL Database and use it to connect from your on-premises server.  
> From the security perspective, it will be as bad as in the previous case. Firstly, you again have a username and password to take care of. Secondly, access management becomes more decentralized and complex as it’s delegated to the application (SQL Server) level. Lastly, you will lack all the identity protection features that an identity provider like Entra ID provides. If those credentials are stolen or misused, you might notice malicious activity in transaction logs long after it happens.

Unfortunately, most tutorials and official documentation describe (and promote in a way) [how to authenticate to Azure resources using client secrets](https://learn.microsoft.com/en-us/dotnet/azure/sdk/authentication/on-premises-apps?#3---configure-environment-variables-for-application), which is similar to using passwords. Thanks to the application registration service design in Entra ID, client secrets are autogenerated complex passwords. Plus, you can [apply policies to your tenants limiting their maximum validity period](https://learn.microsoft.com/en-us/graph/api/resources/applicationauthenticationmethodpolicy), forcing application owners to rotate them on a regular basis.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924481/29e48525-2651-43a8-8a41-95c49993c7b3.png align="center")

On the surface, it looks easy, as you just need to configure your environment variables, and you are good to go. The dark side of that approach is that those client IDs with their secrets can be shared as easily as usernames and passwords, making them vulnerable to misuse, unauthorized exposure, and credential leaks. The temptation and ability to generate never-expiring client secrets (many people don’t want to be burdened with secret rotation) make such app registrations ideal candidates for identity attacks.

[Certificate-based authentication is considered more secure than password-based (client secret) one](https://learn.microsoft.com/en-us/entra/identity-platform/security-best-practices-for-app-registration#certificates-and-secrets). First of all, with certificates, you rely on asymmetric encryption, when only the public part of your certificate is stored along with an app registration, and the private part is secured on your application side. Apart from that, the certificate’s private key can be securely stored in certificate stores, where it can be read only by applications hosted on target machines without the possibility to export and transfer it somewhere else. Also, the complexity of working with certificates and encryption algorithms they use makes them less prominent targets for attacks compared to client secrets.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925255/2af35362-e5e4-417c-ab26-9158944ecc4d.png align="center")

For example, in the diagram above, the client application uses a certificate stored in a Windows Certificate Store to authenticate to the related app registration and obtain an access token. Specialized administrative solutions can manage certificates, so developers don’t even need to touch them. Certificate rotation can performed independently by system administrators. Developers might not even have access to those certificates and can only use them in a controlled manner on managed devices.

## Case 3. Use managed identity with Azure Arc-enabled servers

The complexity of using certificate-based authentication also makes it harder to adapt, especially when you don’t have the capacity or expertise to manage those certificates efficiently. So, what can we do in that case?

What if I told you that there is a way to use managed identities even for servers hosted outside of Azure? By [onboarding your on-premises server, AWS EC2 instance, or any external machine to Azure Arc](https://learn.microsoft.com/en-us/azure/azure-arc/servers/deployment-options), you effectively create a logical representation of it in your tenant, which you can use to manage that resource from Azure and provide it access to other Azure resources using its service principal. The Azure Arc agent running on such machines is responsible for maintaining the connection to the Azure cloud and providing access to that machine authentication context. Applications running on an Azure Arc-enabled server can [use that authentication context to access Azure resources the same way if that server was hosted in Azure and you used its managed identity for authentication](https://learn.microsoft.com/en-us/azure/azure-arc/servers/managed-identity-authentication).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925884/e8632b43-8d27-4687-9677-aa3cb9fcbff1.png align="center")

Although onboarding your servers to Azure Arc has an initial overhead, after the servers are connected, you can leverage their managed identities the same way you do with Azure VMs. For organizations with many on-premises resources, this can be a lifesaver when planning integration with Azure resources.

## Case 4. Use AWS Secrets Manager, AWS Certificate Manager, or analogs to store your credentials

You might ask, okay, what do I do if I need to connect to Azure resources from another cloud and managed identity is not an option for me? In that case, using cloud services specifically designed to store and retrieve sensitive information like secrets, keys, and certificates would be security-wise.

For example, consider your client is hosted in AWS. It uses AWS Lambda functions to retrieve information from an Azure-hosted API.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681926648/eabbf688-e4a5-43bf-99cf-60b15085f296.png align="center")

You can create application registration for your AWS-hosted client in your Azure tenant and generate client credentials. Next, you store those credentials in AWS Secret Manager if it’s a client secret or in AWS Certificate Manager if it’s a client certificate. You [provide access to read those credentials to your Lambda function and then use them](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets_lambda.html) to authenticate to the target Azure resource.

The bottom line is that if you cannot get rid of credentials, you should ensure their safety on the client side. If your Lambda function references those secrets, you don’t even need to know their values. In addition, those secrets can be rotated separately without touching the client application.

In some cases, like connecting from third-party applications or SaaS services, you might have no other option than using client secrets only due to application limitations. However, in such scenarios, you also entrust your credentials handling to that third party. Here, you should carefully consider what permissions you grant to that application in your Azure tenant. If it needs excessive write (or read) access, it definitely should be a red flag for you.

## In conclusion

As you might see from the described use cases, there are plenty of more secure ways to authenticate to Azure services than just using usernames and passwords. Unfortunately, many developers overlook them when designing applications or configuring integration with Azure-hosted solutions. Old habits die hard. Despite the times when applications were mostly self-hosted and connecting to a database located on another server using database user and password are long gone, that legacy still lives in many organizations. The unwillingness to take that extra step to secure your application connections leads to leaked passwords, compromised accounts, and data breaches. So, next time when you deploy an application to production and use usernames and passwords to configure it, remember that [the average cost of a data breach reached an all-time high in 2023 of USD 4.45 million](https://www.ibm.com/reports/data-breach) and increases year over year.

Subscribe 👇 to stay tuned for follow-up posts in this series. Also, share your thoughts on authenticating to Azure services in the comments section!