---
title: "Ghost on Azure: Project Update (Ghost 5, MySQL Flexible Server, Private Link, RBAC for Key Vault, App Service access restrictions to Front Door)"
seoDescription: "Discover the latest updates on deploying Ghost 5 on Azure, integrating enhanced security features like MySQL Flexible Server and Azure Private Link"
datePublished: Tue Nov 12 2024 09:40:51 GMT+0000 (Coordinated Universal Time)
cuid: cm5b9gsfz000409l8bhltbejv
slug: ghost-on-azure-project-update
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736867442838/652580ec-bb95-453e-bb86-1c09ccc32652.png
tags: azure, ghost, containers

---

It has been a while since I last updated my [Ghost on Azure project](https://andrewmatveychuk.com/tag/ghost-tag/), and many changes have been introduced to Azure services during that time. I decided to use the break in my work to update the project deployment templates to include those changes and to use it as a learning opportunity to catch up on new cloud service features.

> [Ghost on Azure](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers) is a one-click Ghost deployment on Azure Web App for Containers. It’s written as a Bicep template, spinning a [custom Ghost Docker container](https://github.com/andrewmatveychuk/docker-ghost-ai) on Azure App Service, which uses Azure Database for MySQL as a backend. It can be a good starting point for anyone wanting to self-host the Ghost platform on the Microsoft Azure cloud. Plus, it leverages a lot of Azure-native services and their features. Thus, it can be considered a showcase of their practical usage.

The project started as a simple multi-container deployment and later transformed into a comprehensive solution using PaaS services such as Azure Key Vault, Front Door, and Web Application Firewall. Now, we are focusing on further security improvements and migrating from deprecated Azure services.

## New Ghost 5 container image

I use a [custom Ghost Docker image](https://github.com/andrewmatveychuk/docker-ghost-ai) in this project, which is based on [the official Ghost Alpine image](https://hub.docker.com/_/ghost/) and extended to [support Azure Monitor Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview). I’ve updated it to Ghost 5 and removed explicit database connectivity check with [wait-for-it](https://github.com/vishnubob/wait-for-it), as it has not been very reliable in testing MySQL server readiness to accept connections. As a new instance of Azure Database for MySQL takes some time to be up and running, Azure App Service hosting the container restarts it a few times until the database backend is ready and the Ghost app can connect to the server to create its database.

> **Note.** The initial deployment might take a few minutes before the Ghost container successfully starts and is ready to serve the content.

## MySQL Flexible Server

[The initial project configuration](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers/) used a multi-container deployment, placing a MySQL container alongside the application container running Ghost using [Docker Compose](https://learn.microsoft.com/en-us/azure/app-service/configure-custom-container?pivots=container-linux&ref=andrewmatveychuk.com&tabs=debian#docker-compose-options). Later, [I switched to a single container deployment and used Azure Database for MySQL for the database](https://andrewmatveychuk.com/how-to-connect-to-azure-database-for-mysql-from-ghost-container/), as the support for multi-container deployment on App Service degraded and became unstable. Recently, [Microsoft deprecated Azure Database for MySQL – Single Server](https://learn.microsoft.com/en-us/azure/mysql/migrate/whats-happening-to-mysql-single-server), and I needed to migrate my project to MySQL – Flexible Server.

As Ghost 5 supports MySQL version 8 as a primary database option, I also switched from previous version 5, which was used with the MySQL – Single Server, to that version with the Flexible Server deployment. However, the most notable change in the context of this project is probably [that Azure Database for MySQL - Flexible Server now enforces using encryption connections](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/how-to-connect-tls-ssl) only by default.

I think using encryption in transit, like connecting to a database, is a good thing, even if the data transfer happens in the internal network. It fully reflects the core principles of [Zero Trust Architecture](https://www.microsoft.com/en-us/security/business/security-101/what-is-zero-trust-architecture), as a malicious actor might also operate inside your network. The tricky part was [configuring the encrypted connection options in the MySQL client used by Ghost](https://sidorares.github.io/node-mysql2/docs/examples/connections/create-connection#createconnectionconfig--ssl). Putting [the public certificate used by MySQL - Flexible Server](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/how-to-connect-tls-ssl#download-the-public-ssl-certificate) into the container image or onto a file share wasn’t a good option and would introduce more unwanted dependencies. So, I ended up [putting the content of that public certificate into an environment variable](https://ghost.org/docs/config/#database) [using multi-line string support in Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/file#multi-line-strings):

%[https://gist.github.com/andrewmatveychuk/7113bfa8d6f89fb558ef51baeadaccaa] 

## Azure Private Link

Another notable change is that I decided to leverage [Azure Private Link](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/concepts-networking-private-link) to further restrict access to the database server at the network level and completely block access to it over the public network.

Configuring [Azure Private Link and private endpoints for Azure services](https://learn.microsoft.com/en-us/azure/private-link/availability) might seem like a daunting task at first, but when you grasp the core concepts of how it works under the hood, it should become your no-brainer option to reduce the attack surface of your cloud infrastructure. Yes, it requires a good knowledge of core network concepts, understanding how DNS resolution works, and configuring corresponding private DNS zones using Azure Private DNS zones or your other existing DNS services. Plus, it adds complexity to your deployments and requires extra work to automate its configuration using Bicep or Terraform templates. However, from the security standpoint, I would name locking down network access to your cloud resources the number one recommendation after setting up proper authorization and encryption controls.

Here is how the project network topology looked like before using Azure Private Link:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922736/58ddd6b9-856e-477b-9b49-ea0d1a175449.png align="center")

Communication between the app service and dependencies happened over their public endpoints. The backend services enforced encrypted connections. Plus, you could put some restrictions using firewalls on their public endpoints to limit access from other Azure services only. Still, the information was traversing over the Internet, and any misconfiguration of service firewall rules could expose them to external attacks.

After [configuring private endpoints for Azure Database for MySQL](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/concepts-networking-private-link), [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/private-link-service) and [Storage](https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints), and configuring [virtual network integration for App Service](https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration), all traffic to backend services is isolated in a private [Azure Virtual Network](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923622/40658a88-05f3-4f12-93a0-22cb0f9c680b.png align="center")

The connectivity to the public endpoints of backend services is completely locked down, making it much easier to control and enforce at the enterprise scale using Azure Policy. Now, you don’t have to validate and control myriads of firewall rules on various Azure resources.

As you can now integrate App Service with a virtual network [even on the Basic tier](https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration#limitations), Azure Private Link and private networking in Azure became even more accessible. When your App Service is integrated with a virtual network, it can connect to other services deployed in that network, like private endpoints, without going over the public network. Plus, you can apply all other security measures available in Azure Virtual Network to further segment and restrict network access using subnets, Network Security Groups (NSG), network policies for private endpoints, etc.

[Azure Private DNS zones](https://learn.microsoft.com/en-us/azure/dns/private-dns-privatednszone) for hosting private DNS zones for Azure Private Link service also greatly simplifies its usage, as it automates the creation of corresponding DNS records for your private endpoints.

## Azure Key Vault role-based access (RBAC)

The next improvement is related to configuring access to Azure Key Vault using Azure roles instead of legacy Key Vault access policies. In my opinion, it greatly simplifies access management at scale, as you no longer need to control access at two different levels, and you can configure access to both management and data plane using Azure built-in and custom roles.

From the Bicep code perspective, we now create a separate Azure role assignment resource:

%[https://gist.github.com/andrewmatveychuk/96bdaab91d7793b684befa4d8325321e] 

Although you can configure [Microsoft Entra authentication for Azure Database for MySQL](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/how-to-azure-ad) and leverage App Service managed identity for authorization, it seems that authentication using managed identities is not supported by the MySQL client library used by Ghost. So, I still rely on MySQL authentication and have to use Key Vault to securely store the database password used to connect to the Ghost database.

Also, when I tried to migrate the Storage account used to host the file share as persistent storage for the Ghost container to the role-based access model, I faced a similar issue, as [that scenario is not supported by custom-mounted storage](https://learn.microsoft.com/en-us/azure/app-service/configure-connect-to-azure-storage?tabs=basic%2Cportal&pivots=container-linux#limitations).

## Azure Front Door Standard and App Service access restrictions

[The initial project version](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers/) used Azure CDN for traffic offloading. Later, I added [an option to deploy the solution with Azure Front Door](https://andrewmatveychuk.com/ghost-deployment-on-azure-security-hardening/), which used a managed Web Application Firewall (WAF) policy for inbound traffic inspection and site protection. Those Azure services, now renamed as classic ones, are [scheduled for retirement in 2027](https://azure.microsoft.com/en-us/updates/azure-front-door-classic-will-be-retired-on-31-march-2027/). Microsoft released a comprehensive guide on [migrating from the Azure Front Door (classic) to the Standard/Premium tier](https://learn.microsoft.com/en-us/azure/frontdoor/tier-migration). I would consider that service update a breaking change, as service features don’t map one-to-one between the classic and new service offerings. Plus, the pricing model of the new offerings is quite different, which requires careful consideration and [cost estimation for your specific use case](https://learn.microsoft.com/en-us/azure/frontdoor/understanding-pricing).

Having said that, I’ve removed the option to deploy the solution with deprecated Azure CDN (Microsoft CDN (classic)) and updated the configuration to deploy Azure Front Door Standard as a more reasonably priced service for such a project. Unfortunately, the managed WAF policies are now supported only by the Premium tier, so Azure Front Door Standard now works more like a CDN, but you can still enhance it with custom WAF policies.

Apart from that, the Azure App service now supports [more targeted access restrictions](https://learn.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions?tabs=azurecli#restrict-access-to-a-specific-azure-front-door-instance). In addition to restricting access to your Azure App service using the Azure Front Door service tag, you can now narrow it to a specific Front Door instance with HTTP headers. The Bicep template part for configuring such restrictions might look like the following:

%[https://gist.github.com/andrewmatveychuk/8f95d0064ab33434c0f717cb0846b225] 

The master project template still contains [conditional logic](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/conditional-resource-deployment) to deploy the solution with an Azure App Service as a public endpoint or use an additional Front Door profile to serve the incoming traffic to your Ghost on Azure deployment.

## Other minor tweaks

In addition to updating Azure Resource Manager API versions for resources, removing discontinued pricing tiers for some services, and updating categories for Azure Monitor Logs diagnostic settings, I’ve also reduced the number of output values passed between the Bicep modules in the master deployment template and used [references to existing resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/existing-resource) whenever possible, as it provides more flexibility in referencing their properties in other resources. For example, instead of passing (aka exposing) Storage account access keys through output values, you can just reference them using the corresponding function on the referenced resource:

%[https://gist.github.com/andrewmatveychuk/ba3046eaf9203f40a132e3acf20ca366] 

> For the complete deployment configuration details, check [the source code in my GitHub repo](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers).

## To be continued

As you might have noticed, a few design decisions in that project originated from overcoming Azure service limitations. I think hosting a containerized app on Azure App Service is still quite limited, not production-wise, as it probably creates more challenges than solves them. I’m considering trying out Azure Container Apps, which will be explored.

Another planned modification is configuring the Azure Private link for Azure Monitor components used in the solution. It’s different from private links to other Azure services, so I plan to explore its specifics in a separate post using my Ghost on Azure project as a playground for that implementation.

Have you used Azure Container Apps or Azure Monitor Private Link Scopes in your projects? What was your experience with them? Please share your thoughts in the comments 👇