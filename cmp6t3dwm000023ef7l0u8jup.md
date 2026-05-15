---
title: "Ghost on Azure: Container Apps"
datePublished: 2026-05-15T11:00:00.000Z
cuid: cmp6t3dwm000023ef7l0u8jup
slug: ghost-on-azure-container-apps
cover: https://cdn.hashnode.com/uploads/covers/6603d3957f11f335976b1c48/e6f967df-ae6f-4c40-92af-a1bd94e4f63f.png
tags: azure, ghost, containers, bicep

---

If you have been following my experiments for hosting and [running Ghost on Azure](https://andrewmatveychuk.com/tag/ghost), you might recall that I tried hosted it on [Azure Web App for Containers](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers) first, using the Docker Compose for running a MySQL database as a sidecar, then I switched to [a single container deployment using Azure Database for MySQL](https://andrewmatveychuk.com/how-to-connect-to-azure-database-for-mysql-from-ghost-container), after that, I rebuild it to use [private endpoints, Azure Key Vault, and put it behind Azure Front Door with WAF](https://andrewmatveychuk.com/ghost-on-azure-project-update) enabled.

> **Disclaimer.** Please treat everything below more like technical experimentation than a practical suggestion for hosting the Ghost platform on Azure, especially if you intend to use it a personal blog. I'm using Ghost here purely as a simple web application that requires dependencies such as databases, file storage, secret management and logging. If your goal is to run a Ghost blog specifically, there are better options, in my view. For example, you can [self-host it on DigitalOcean](https://andrewmatveychuk.com/how-i-run-my-blog) or go with a [fully managed Ghost Pro option](https://andrewmatveychuk.com/moving-to-ghost-pro) to get rid of the platform management overhead.

Lately, I decided to refactor that setup and rehost the Ghost container on Azure Container Apps instead of App Service, with the primary intent of testing that new Azure service offering and gaining a better understanding of its pros and cons.

> For the complete solution code containing all required Bicep templates, please check my [azure.ghost-web-app-for-containers](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers) repository on GitHub.

# Ghost on Azure Web App for Containers

Before jumping into Container Apps, let's take a quick look at the overall infrastructure architecture, with Ghost running on Azure App Service.

![](https://cdn.hashnode.com/uploads/covers/6603d3957f11f335976b1c48/75b08b65-edee-46e8-a354-d8e081f4bfbe.png align="center")

As you can see from the diagram above, the deployment template, which I created previously, provisions a virtual network and injects connection endpoints into it for web app dependencies such as Azure Database for MySQL Flexible Server, Azure File Share on a Storage account, and Azure Key Vault. The App Service hosting the Ghost container is configured with a VNet integration, so the application can reach all its dependencies via the private network. The App Service is prefaced with an Azure Front Door Standard profile.

> Previously, the deployment used the Azure Front Door classic service, which supported managed Web Application Firewall (WAF) policies even on the back-then Standard tier. With the retirement of the classic service, I switched to [the new Azure Front Door Standard option, which became more of a CDN with limited WAF support](https://learn.microsoft.com/en-us/azure/frontdoor/front-door-cdn-comparison).

Traffic to the App Service is allowed only from the Front Door profile, using App Service access restrictions and the profile's unique identifier.

For more detailed information about that setup, I suggest checking [my previous post](https://andrewmatveychuk.com/ghost-on-azure-project-update).

# Ghost on Azure Container Apps

Now, let's see how that architecture changes if we want to host our web app container on [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/overview).

![](https://cdn.hashnode.com/uploads/covers/6603d3957f11f335976b1c48/3b934758-2eb8-465e-974d-7c2a5a24feae.png align="center")

From a network perspective, the application dependencies and their network part remain almost untouched. The key difference is that app VNet integration is performed at the Container App Environment level, not at the Container App level, unlike App Service. Plus, the network topology becomes more complex, especially if you want to protect your web application with WAF.

When you deploy your Container App Environment (aka App Service Plan for Container Apps), you can [choose whether it is going to be accessible from the public network or not](https://learn.microsoft.com/en-us/azure/container-apps/networking?utm_source=copilot.com). You can also configure each Container App ingress to accept external traffic or to accept connections only from the internal container network. Since Container Apps don't have anything similar to App Service access restrictions, the closest (and possibly the most secure and expensive) option to expose your application to the Internet is to [preface it with an Azure Front Door Premium profile](https://learn.microsoft.com/en-us/azure/container-apps/how-to-integrate-with-azure-front-door), which supports private links.

> Note. You can also [secure your ingress traffic for Container Apps using Azure Application Gateway](https://learn.microsoft.com/en-us/azure/container-apps/waf-app-gateway?tabs=default-domain). The choice of Azure Front Door in my case is merely subjective to minimize overall infrastructure changes.

The Container App Environment is deployed with an internal load balancer, meaning it has no public endpoints and can be reached only from its internal (integration) network. The load balancer is provisioned automatically and managed by Microsoft. It is used to reach the underlying AKS cluster powering your Container App Environment and abstract the internal network routing from us. Azure Front Door, in turn, is provisioned with a private link pointing to that internal load balancer. Under the hood, another managed network is used to create a private endpoint for your Azure Front Door instance and connect it to the load balancer via Private Link. That way, traffic from users goes to Azure Front Door, which can inspect it with configured WAF policies and route it to the Container App Environment via the private network.

Now, let's dive into the web app configuration on Container Apps itself.

# Configuring a Container App

Before creating a Container App, you need to have a [Container App Environment](https://learn.microsoft.com/en-us/azure/container-apps/environment), which is similar to an App Service Plan in terms of providing underlying infrastructure resources for your app.

## The Environment

The Bicep template for provisioning the environment is pretty straightforward:

%[https://gist.github.com/andrewmatveychuk/22edae5fa470341634426fdfa09afc31] 

The tricky part is that, for our (advanced) setup with private networking and Azure Front Door private link, you must first [provision a virtual network and configure an appropriate subnet to be delegated to the environment](https://learn.microsoft.com/en-us/azure/container-apps/custom-virtual-networks). Also, the environment type must of Workload profile.

> Please note that [Consumption-only environments have been deprecated](https://learn.microsoft.com/en-us/azure/container-apps/environment-type-consumption-only). Plus, they lack many of the networking features available in Workload profile environments. Many old guidelines for Container Apps on the Internet refer to legacy Consumption-only environments, so please check [the official documentation for the latest guidance](https://learn.microsoft.com/en-us/azure/container-apps/environment#types). Otherwise, you might spend additional hours (as I did 🤪) troubleshooting and trying to understand why something doesn't work.

As you might see from the code snippet, the virtual network configuration for the environment is set to internal, so it won't have a public endpoint. Also, connecting to Container App environments using Private Link is only supported in that configuration.

Another essential component is [configuring storage for our environment](https://learn.microsoft.com/en-us/azure/container-apps/storage-mounts). Connected storage is configured for the environment, not an app, so if you have a multi-container application or need to connect to external storage, all your applications hosted in that environment can use the same connections.

%[https://gist.github.com/andrewmatveychuk/eb0bf7194f2f2a78df8d296c670a6d06] 

Technically, Container App Environments support both SMB and NFS file shares as persistent storage for your Container Apps. However, I found using Azure Files SMB file shares with Container App Environments very unreliable: a slight difference in how file permissions are managed between Linux and SMB shares would completely prevent a containerized application from accessing or writing to a connected SMB share. The only stable solution is to create and connect an NFS file share, which immediately increases storage costs, as [NFS file shares are only supported on premium storage tiers](https://learn.microsoft.com/en-us/azure/storage/files/files-nfs-protocol#performance) with a preprovisioned capacity.

Another caveat is that the HTTPS enforcement at the Storage account level must be explicitly disabled. NFS Azure file shares [encrypt data in transit on the protocol level](https://learn.microsoft.com/en-us/azure/storage/files/encryption-in-transit-for-nfs-shares).

> Note that there are some nuances in how encryption in transit for NFS is handled for new and existing Storage accounts. Please check [the official documentation](https://learn.microsoft.com/en-us/azure/storage/files/encryption-in-transit-for-nfs-shares#enforce-encryption-in-transit) for recent updates.

The last Container App Environment dependency, the Log Analytics workspace, is probably the easiest and most straightforward. It is used to collect all logs and metrics for your Container Apps.

## The Container App

Now, let's check the configuration of the Container App itself:

%[https://gist.github.com/andrewmatveychuk/c8d43ade3b2b009149ecbdc49032d9c3] 

The first thing you might notice is that the resource configuration parameters for Container Apps resemble those of a deployment in Kubernetes: you define containers, volumes, mounts, replicas, secrets, etc. On the one hand, it makes it easier for those familiar with Kubernetes to understand a Container App configuration straight away. On the other hand, it is not much simpler than a Kubernetes deployment config: even with the minimal required setup, you still need to define and provide a lot of parameters to run your (container) app.

Secondly, external traffic to the app is allowed, meaning that it can be reached from our private virtual network through the managed load balancer, but not from the Internet. The container that exposes the port is the one receiving the incoming traffic. Only one container can be exposed, and in multi-container deployments, all other containers can communicate only via the internal container network.

The container status can be automatically checked by container probes, similar to Kubernetes:

%[https://gist.github.com/andrewmatveychuk/44207dfcf23306dc1b5fb87d429b5e3d] 

In the case of Ghost, the web app doesn't expose any meaningful health statuses, so the sample health checks are very basic.

Container volumes and mounts are configured similarly to Kubernetes deployments:

%[https://gist.github.com/andrewmatveychuk/cf94a074381af57a88be1019b94a31f1] 

The storage type, which is the NFS file share, must match the one defined in your mounted Container App Environment storage.

To inject any secrets to be used by your Container App, you can (and should) pass them as references to secret values stored in an Azure Key Vault:

%[https://gist.github.com/andrewmatveychuk/bcb90d429ef32ca1455f3a7fea55f54d] 

There are a few nuances with [accessing and handling Key Vault secrets with Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/manage-secrets):

*   To access a Key Vault secret, your Container App must have the appropriate permissions, e.g., the Key Vault Secrets User role.
    
*   The app also needs an assigned identity with the required access role for the Key Vault. In my case, I'm using a user-assigned managed identity to avoid the chicken-and-egg problem when automating resource provisioning with Bicep templates.
    
*   The container secret name must contain only lowercase alphanumeric characters and hyphens, and must start and end with an alphanumeric character. Otherwise, your deployment will fail.
    
*   You specify the identity at both the Container App level and at the container (deployment) level. That is because you can assign multiple identities to a Container App and specify which identity to use to access a specific secret at the container level.
    

Lastly, there are container variables:

%[https://gist.github.com/andrewmatveychuk/a7b7f6cc00e87a79d8a771de1a21c40e] 

They are specific to your application and represented as a key-value array. In my case, they are specific to Ghost for configuring the website settings and database connection. For the latter, the secret reference name for the database password must match the corresponding container secret name, which is injected in the container runtime as an environment variable.

## Other dependencies

Now, let's look at the remaining dependencies for our sample web app.

### The Database

Since my previous Ghost deployment on Azure App Service already used [Azure Database for MySQL Flexible Server](https://andrewmatveychuk.com/ghost-on-azure-project-update#heading-mysql-flexible-server) to host the Ghost database, I just retained that configuration. The only change required was updating the public certificate used to establish [an encrypted connection to the database server](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/security-tls-how-to-connect#download-the-public-ssl-certificate), as Microsoft rolled out certificate updates on the service side.

### The Azure Front Door profile

As I mentioned earlier, the Front Door profile must be provisioned on the Premium tier to support private link connectivity. [The Front Door and WAF configuration is pretty standard, and the only unobvious part is the private link setup](https://learn.microsoft.com/en-us/azure/container-apps/how-to-integrate-with-azure-front-door):

%[https://gist.github.com/andrewmatveychuk/15485a9d9131533f2953746ca8ef9a59] 

The Private Link is configured for a specific Front Door origin, and it must target our Container App Environment and be deployed in the same location. The `groupId` property name is a bit misleading, as it actually refers to the actual resource type to be connected by Private Link, which is `managedEnvironments`, inferred from the Container App Environment resource type.

> For the complete solution code containing all required Bicep templates, please check my [azure.ghost-web-app-for-containers](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers) repository on GitHub.

# Deployment considerations

Please note that some services in that configuration can take tens of minutes to deploy, depending on resource availability in a particular Azure region. The longest-deployed resources are Azure Database for MySQL and the Container App Environment. In my tests, the whole deployment could run for 40 minutes or so, primarily due to the long provisioning time for Container App Environments.

That long provisioning time might also result in timeouts, when Azure Resource Manager would simply terminate the Container App Environment deployment due to no response from the service backend. Unfortunately, due to some bugs (or features) of the Container App Environment API, the terminated deployment would lock the integration subnet as already delegated to that service, effectively making incremental deployments impossible. On the next attempt, you would receive an error about the subnet being already in use by another service/instance. Manually removing the delegation hadn't reliably resolved the conflict, so I had to delete the whole resource group and start a fresh deployment, since too many resources (like private endpoints) also depend on the virtual network.

If you encountered a similar issue with long Container App Environments deployments and overcame it, please let me know 💬

## Post-deployment steps

There are few things that cannot be or are hard to automate in that deployment:

*   Firstly, you need to manually approve a connection via Private Link from the Front Door, since it essentially accepts a private link from an externally managed environment. That's just how Private Link works, and you cannot automatically approve it using Bicep templates. Technically, it's still possible to run a post-deployment script that executes the same API call you initiate from the Azure Portal when you accept the private link connection. However, I think such automation is not very reasonable from a security perspective. Plus, the automation effort might not be worth it, since spinning up Azure Container Environments and Front Door profiles isn't something you would usually do dynamically or at scale (more on this later).
    
*   Secondly, because of the nuances of Ghost configuration, you must specify the correct canonical URL for accessing the website. That URL becomes known only after the Front Door profile deployment, and you need to update the environment variable in the deployed Container App, which creates a new container app revision (that's how Container Apps operate). If you use a custom domain for your Front Door profile, that becomes less of an issue because you can programmatically construct the correct URL in advance.
    

# My thoughts about the practical application of Azure Container Apps

If you managed to read to the bottom of this post, you might get the impression (as I did) that deploying and managing Azure Container Apps is fairly complicated and nuanced. The initial investment to get your application up and running in a relatively secure and enterprise-ready setup is pretty high. Yes, there are enough official and unofficial sample templates to deploy and run a Container App, but the Container App is just one piece of a puzzle. You would rarely run it in isolation. Even a small web application like Ghost usually requires a database, network, security, and other components, and that's where it all becomes much more complicated.

Another overhead is that the Container App Environment and Container App configuration are not as easy as advertised. You need to dive into the nuances of how those resources are provisioned, how they integrate with other Azure services, their limitations, and how the promise of running your container apps easily with them limits your architectural and technical flexibility. That knowledge is not transferable to other platforms or cloud providers.

Apart from that, compared to Kubernetes and its managed cloud versions, where you can take your Helm charts or native deployment templates and redeploy your application on another cluster, which can be hosted elsewhere, app deployment templates created for Container Apps are not so easily migrated without (sometimes) significant refactoring.

The deployment issues I mentioned earlier also raise questions about Container App's positioning as a scalable cloud service. Similar to Kubernetes clusters, Container App Environments are more of a persistent infrastructure layer you use to run your ephemeral, autoscalable container-based workloads. However, if we consider them from that persistence perspective, where you provision and fine-tune them semi-manually and not something you deploy and scale fully automatically, what is the benefit of using them over traditional Kubernetes? I expected more elasticity from Container Apps, which they don't deliver in their current form.

# In conclusion

Container Apps provide a lot of out-of-the-box functionality, so you can focus more on your application rather than on managing underlying infrastructure. Still, I see them as a very niche solution that can benefit you in some cases, but I wouldn't call it one that makes running container apps easy. To me, it's still very nuanced, vendor-locked, and complex to suggest as a go-to approach for running container apps in Azure. The issue with Container App Environment deployments and scaling also doesn't add brownie points when you consider them for enterprise use cases, where reliability becomes a deciding factor.

If you asked me about good options for running container apps in Azure, I would most likely still go with Azure Web Apps for Containers or Azure Kubernetes Service, provided there are no specific requirements or constraints that limit the choice. The former is much simpler to configure and deploy, it autoscales reliably, and is a solid choice for simple or single-container web app deployments, especially when you just want to pack and rehost some legacy app without refactoring it. The latter gives you much more control and flexibility with your container infrastructure, widely used, and has a far better ecosystem to support all your needs. To me, the benefits of using Azure Container Apps simply don't outweigh all its drawbacks.

What is your experience with Azure Container Apps? Have you found them beneficial for your use cases? Did you run them with production workloads? Or are they something you would not prefer to use? Please share your thoughts in the comments 💬