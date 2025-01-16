---
title: "Ghost deployment on Azure: Security Hardening"
seoDescription: "Secure your Ghost deployment on Azure with Key Vault, Azure Front Door, and Web Application Firewall for enhanced protection and performance"
datePublished: Mon Feb 15 2021 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b6d5od00010al8bv07ge8h
slug: ghost-deployment-on-azure-security-hardening
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737043040009/6b9c3ffa-14e3-4be2-aa91-1a0d572b1f3f.png
tags: azure, security, ghost, how-to, system-design

---

In [the first part of this series](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers), I wrote about running Ghost on Azure Web App for Containers. As I promised last time, we will explore some security improvements to the original deployment configuration here.

## Keeping your passwords secret with Azure Key Vault

It’s not a secret that you should store and handle passwords, API access keys, certificates, etc., in a secure way. The suggested approach of doing so in Azure is using [Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/), which is a service specifically designed to store and retrieve sensitive information securely.

Despite Azure Key Vault being promoted in almost every design document produced by Microsoft, I still see many cases when developers just put credentials like database passwords in the application settings or have them in plain text in the connection strings. Besides, some time ago, the Azure product team introduced [hidden values for the application settings](https://docs.microsoft.com/en-us/azure/app-service/configure-common#show-hidden-values) by default so that nobody gets your password by [shoulder surfing](https://w.wiki/Chyd). To my mind, it made the security awareness even worse as that ‘Hidden value’ label usually deceives people and makes them think that their passwords are secure:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922725/dce15d26-6ec2-4e33-8f72-521a1a552e71.png align="center")

In reality, everything you store in the application settings and connection strings is *not safe* as it can be accessed (and read) by anybody having read permissions on that resource, which can be dozens of people in your organization and even beyond it.

I know all that might sound like pretty basic stuff for an experienced cloud engineer or architect, but it is still worth mentioning to increase people’s awareness.

For the sake of simplicity, in the initial version of my deployment configuration for [Ghost on Azure Web App for Containers](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers), I omitted the application's security aspect and placed the database password provided during the deployment process into the application settings. Now, it is time to fix that ‘vulnerability.’

The Azure Key Vaults deployment is well documented, and there are plenty of [sample ARM templates for it](https://github.com/Azure/azure-quickstart-templates). First, we will need a [Key Vault resource](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) in your configuration. Secondly, we should create a [Key Vault secret](https://docs.microsoft.com/en-us/azure/key-vault/general/about-keys-secrets-certificates#object-types) to store the Ghost app database password to authenticate to the MySQL database. Thirdly, we must configure an [access policy](https://docs.microsoft.com/en-us/azure/key-vault/general/secure-your-key-vault#data-plane-and-access-policies) granting the app permission to read (get) secrets from the key vault.

%[https://gist.github.com/andrewmatveychuk/f22abcc32247f081ed168b8536788d1e] 

We will use a [system-assigned managed identity](https://docs.microsoft.com/en-us/azure/app-service/overview-managed-identity) created for our web app for configuring the access policy. Referencing the identity dynamically during resource provisioning is a bit tricky and requires some knowledge of [the ‘reference’ function](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-resource?#reference) and [Azure REST API](https://docs.microsoft.com/en-us/rest/api/appservice/webapps/get#site).

Finally, we should [reference our Key Vault secret](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references) containing the database password in the application settings of our web app:

%[https://gist.github.com/andrewmatveychuk/948e7790b0d434b24bac537963afd4d8] 

If everything was configured correctly, you should see a green checkmark alongside the app setting, and the reference should be resolved upon the deployment:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923627/e8368ac6-bdb6-4afd-a667-1d233259f4cb.png align="center")

In production deployments, depending on your security requirements, it might make sense to provision and manage key vaults separately from your applications, e.g., putting them in other resource groups with no access to Key Vault resources for application developers. Basically, you can enable the developers and apps to use the secrets without exposing their values. If the secrets were compromised or need to be changed, you would need to update them in a Key Vault only (supposing you don’t reference a specific secret version).

## Securing your app with Azure Front Door and Web Application Firewall (WAF)

[The initial configuration](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers) leveraged [Azure CDN](https://docs.microsoft.com/en-us/azure/cdn/cdn-overview) to improve page load performance and offload traffic from the web app. Although Azure CDN does an excellent job and can even add some security-related features to your app, it is not a firewall that can protect your website from malicious attacks. So, what can we do with that?

Technically, we can put our web app behind an [Azure Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview), which is an OSI layer 7 load balancer with lots of [security features, including traffic inspection and web firewall](https://docs.microsoft.com/en-us/azure/application-gateway/features). Additionally, to ensure the app’s high availability, we might decide to deploy it into multiple regions and route user traffic to a suitable app instance with [Azure Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview). Also, don’t forget about configuring Azure CDN for offloading static content. Sounds quite overwhelming for a simple web app, yeah?

In 2018, [Microsoft announced Azure Front Door service](https://azure.microsoft.com/en-us/blog/announcing-public-preview-of-azure-front-door-service/) as an all-in-one solution for acceleration, load balancing, and securing your web applications. So, basically, instead of designing and maintaining complex infrastructure consisting of many different Azure services, now you can achieve almost the same results by using [Azure Front Door](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview).

Please note that Azure Front Door is neither an evolution nor a replacement for other Azure-based traffic management and load-balancing services. Its relative simplicity comes with a price of less control over the underlying network configuration. There are still cases where you would prefer building a ‘front door’ for your apps with more low-level solutions.

> I am not going here into a detailed explanation of how the Azure Front Door service works or how to configure it. For those of you who already have worked with [Azure Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview), the concepts of frontends, backend pools, routing rules, and WAF should be pretty familiar. If you are new to this topic, I suggest checking out the “[Managing Network Load Balancing in Microsoft Azure](https://andrewmatveychuk.com/refer/microsoft-azure-network-load-balancing-managing)” course by Tim Warner.

Speaking of our deployment configuration for Ghost on Azure Web App for Containers, I created its copy and replaced the Azure CDN resource with an Azure Front Door one. In addition, I defined a [Web Application Firewall](https://docs.microsoft.com/en-us/azure/web-application-firewall/afds/afds-overview) policy and assigned it to the frontend of my Front Door instance. The policy contains only [the default Azure-managed ruleset](https://docs.microsoft.com/en-us/azure/web-application-firewall/afds/afds-overview#azure-managed-rule-sets), which has lots of rules to protect your app from different attacks.

%[https://gist.github.com/andrewmatveychuk/846af90039393a6b94f460d47993b2b6] 

When deployed, the Front Door configuration on the Azure portal will be like the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924204/b9d1d5ea-f36f-41a5-a531-07ac0dcf2ad6.png align="center")

The WAF policy should be linked to the Front Door frontend, and the default rule set should be enabled:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924803/59fa794c-0a77-45f0-8fd8-9cde3829d772.png align="center")

Give a few minutes after the initial app deployment for your web app to warm up and for the Azure Front Door instance to start serving traffic before accessing your Ghost instance at *https://* (frontDoorHostName is specified in the outputs of the template deployment and is something like `ghost-fd-tigs2e7rcquoy.azurefd.net`).

So, now the web app is securely accessible via Azure Front Door, but it still can be reached by its default name provisioned by App Service at `https://uniqeappname.azurewebsites.net`. Luckily, there is a simple solution for that. You can [configure access only from Azure Front Door to your web app by creating an access restriction rule in App Service](https://techcommunity.microsoft.com/t5/azure-architecture-blog/permit-access-only-from-azure-front-door-to-azure-app-service-as/ba-p/2000173). In an ARM template, you would need just add the corresponding setting in the ‘siteConfig’ section:

%[https://gist.github.com/andrewmatveychuk/aaa0f4b05b6c07b6f01fb8be33c8fad9] 

When the restriction rule is in place, you will receive the HTTP 403 status code when trying to reach the web app using its default name at the `azurewebsites.net` subdomain.

Finally, all incoming requests to the web app are coming only through Azure Front Door, which has proper traffic inspection and security protection.

Although using Azure Front Door to publish your web apps can be beneficial, those benefits have their cost tags. In other words, running an application with Azure Front Door will cost you more than running it with Azure CDN. For that reason, I decided to keep both deployment options in [my GitHub repository](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers).

If you have any questions about this topic, please post them in the comments section below 👇, and keep your web apps secure!