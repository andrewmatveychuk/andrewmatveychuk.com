---
title: "How to deploy to different tenants with Azure DevOps"
seoDescription: "Deploy Azure resources across tenants using Azure DevOps. Explore multi-tenant setups to streamline CI/CD processes"
datePublished: Wed Jun 30 2021 07:25:02 GMT+0000 (Coordinated Universal Time)
cuid: cm5b7olnz00030ajmeco2f6kl
slug: how-to-deploy-to-another-tenant-with-azure-devops
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736871106654/2584fb68-9b09-4477-af19-6e5694475a49.png
tags: azure, how-to, azure-devops

---

*Is it possible to deploy to an Azure subscription in another Azure AD tenant with Azure DevOps? How can I configure my Azure Resource Manager service connections in Azure DevOps to point to different tenants? Can I configure multi-tenant deployments with Azure DevOps? I hear those questions occasionally, so let’s try to answer them in this post.*

Although most organizations, especially with centralized IT management, prefer to build and operate their infrastructure within a single Azure AD tenant, there are still enough corner cases when you need to span your deployment process across multiple tenants. For example, some enterprises prefer completely isolating their development/test environments, including identity providers. Others, like managed service providers (aka MSP), usually provide their services to multiple clients and, therefore, have to operate in a multi-tenant environment.

Azure DevOps, a common choice for application lifecycle management when a company mainly works in the Microsoft ecosystem, can be used to set up your CI/CD processes along with underlying infrastructure provisioning in the Azure cloud so that deployments are performed consistently, repeatably, and automatically, establishing the foundation for the Flow in DevOps methodology.

> Here and now on, I refer to [Azure DevOps Services and not to on-premises Azure DevOps Server](https://docs.microsoft.com/en-us/azure/devops/user-guide/about-azure-devops-services-tfs).

Connecting your Azure DevOps organization to an Azure AD tenant usually occurs under the table while onboarding to Azure DevOps services, and creating service connections to Azure subscriptions in the same tenant is pretty intuitive when you follow the default service recommendations. On the contrary, when you need to configure your deployment into an Azure subscription bound to a tenant different from the one used in your Azure DevOps configuration settings, things become a bit trickier. So, let me clear the air first about the common misconception about the relationship between Azure AD and Azure DevOps.

## Azure AD and Azure DevOps

To clarify, [connecting Azure DevOps to Azure Active Directory](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad) has relatively little to do with [Azure Resource Manager service connections](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#sep-azure-resource-manager). That connection just provides a means for your users to authenticate to Azure DevOps using the same credentials they use to log in to Office 365 and other Microsoft services used in the organization.

Azure AD tenant, which is an instance of [Azure Active Directory services](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis), provides cloud-based Identity as a Service (IDaaS) for your organization. From user experience, the integration between Azure AD and Azure DevOps simplifies the configuration of the service connections when speaking of Azure services. For instance, when [creating a new Azure Resource Manager service connection in Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure), users can see all Azure subscriptions they can access that tenant. However, those connections require using separate identities, aka [service principals](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals), to function.

> Be aware that [reconnecting your Azure DevOps organization to another Azure AD tenant](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/change-azure-ad-connection) is a somewhat destructive action. That way, you change the identity provider for Azure DevOps as an application. **It was not designed to help you with multi-tenant deployment scenarios**.

Now, let’s talk about what setups you can use to connect from Azure DevOps to Azure services in another tenant.

## Azure Resource Manager service connection with an existing service principal

When creating an Azure Resource Manager service connection, you can choose [to configure one using an existing service principal](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal). Moreover, that service principal doesn’t have to be in the same tenant your Azure DevOps organization is connected to.

For that scenario, you need to know the ID and name of the target Azure subscription (or management group if you plan to deploy in that scope), a target Azure AD tenant ID and a service principal ID with its key or certificate in that tenant to use in the connection. Of course, in a fresh new environment, you must [create that service principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell) first. Nevertheless, that approach is well-documented and straightforward.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922729/caa5c3ba-0e9d-4458-9074-ab6169932185.png align="center")

The main advantage of this approach is that the setup time is minimal, plus the configuration is performed only in two places – a target Azure AD tenant and your Azure DevOps service connection. Still, that initial simplicity has its maintenance cost. When you have only a few service principals in a few different tenants to maintain, the overhead of refreshing expired service principal keys or certificates and updating the corresponding service connections is insignificant. However, when you have hundreds of tenants in management and even more Azure DevOps service connections to them, their maintenance might become a never-ending and time-consuming routine.

> By default, when you [create an Azure Resource Manager service connection using automated security](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#create-an-azure-resource-manager-service-connection-using-automated-security), Azure DevOps creates an app registration with a secret that’s valid for two years. When creating an app registration with service principal manually to use for Azure DevOps connection, you can increase that expiration period or even set the secret to never expire so that you don’t need to go back and update them. However, I would not recommend doing that from a security perspective.

## Azure DevOps service connections with Azure Lighthouse

Another approach to connecting Azure DevOps to resources in a different tenant is using [delegated access via Azure Lighthouse](https://docs.microsoft.com/en-us/azure/lighthouse/concepts/architecture). Although it was not explicitly designed to address the challenge of deploying to different or multi-tenant environments, Azure Lighthouse provides excellent options for unifying access experiences in multi-tenant environments.

Conceptually, when you [onboard Azure subscriptions (or specific resource groups) to Azure Lighthouse for delegated management](https://docs.microsoft.com/en-us/azure/lighthouse/how-to/onboard-customer), you provide particular sets of permissions to them for identities from another (management) tenant. Those identities can be individual user accounts, security groups, or service principals. For example, you can configure delegated access to another tenant for specific security groups in your tenant, [which is a suggested practice](https://docs.microsoft.com/en-us/azure/lighthouse/concepts/recommended-security-practices#assign-permissions-to-groups-using-the-principle-of-least-privilege), and include a service principal used by Azure DevOps service connections into one or a few of those groups to get the permissions you need for your deployments.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923713/45d5ce48-d70a-45a0-8998-f39dadbb9689.png align="left")

In that scenario, you still need to manually create a service principal for use in Azure DevOps service connections, but you provision it in your (management) tenant that is connected to your Azure DevOps organization.

> Note that [with Azure Lighthouse, you cannot delegate the Owner role or manage access to resources in delegated tenant](https://docs.microsoft.com/en-us/azure/lighthouse/concepts/cross-tenant-management-experience#current-limitations) except for limited use in assigning roles to managed identities. If your deployment scenarios require some automated access configurations, you might consider the previously described approach or the next one.

The benefit of this configuration is that you can create a single or a few service principals that you can reuse when connecting to different tenants, effectively making your architecture more scalable. You will need to spend less time in the future on regenerating the keys and updating the connections. What is more, when the target tenant is offboarding from external management via Azure Lighthouse, that will remove all the accesses to resources in that tenant that were granted to external Azure DevOps service connections, so you don’t have to worry about revoking access on some hell-knows-what service principals.

Implementing such a setup requires more time to initialize, provide delegated access via Azure Lighthouse, and maintain delegations relevant to your deployment scenarios.

## Azure DevOps service connections and Administer on Behalf of (AOBO)

[Administer on Behalf of](https://docs.microsoft.com/en-us/azure/lighthouse/concepts/cloud-solution-provider#administer-on-behalf-of-aobo), or AOBO for short is a functionality of [the CSP program](https://docs.microsoft.com/en-us/partner-center/csp-overview) that makes all users with the Admin Agent role in your tenant owners of the subscriptions that you create through the CSP program. So, technically you can add your service principal(s) provisioned for Azure DevOps in your tenant to [the Admin agents group](https://docs.microsoft.com/en-us/partner-center/permissions-overview#manage-commercial-transactions-in-partner-center-azure-ad-and-csp-roles) to grant them access to the subscriptions in managed tenants.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924413/3f6c153c-e4af-435d-b171-cb4ae7bc71b7.png align="center")

That approach is quite similar to the delegated access via Azure Lighthouse, but it’s less flexible. Basically, it follows the all-or-nothing principle. Once identity is included in the Admin agents group, it has complete access to all resources in all subscriptions managed by a CSP. While with Azure Lighthouse, you have an opportunity to configure more granular and least-privileged access.

On the positive side, as the members of the Admin agents group are granted the Owner permissions to the subscriptions, they are free of Azure Lighthouse’s limitation of controlling access to managed resources.

Obviously, configuring Azure DevOps service connections via AOBO is limited only to the subscriptions in your CSP program. In other words, it’s not suitable for deploying into subscriptions that aren’t linked to your CSP organization.

The three mentioned approaches to deploying into a different tenant(s) with Azure DevOps aren’t exclusive. Azure DevOps as a platform can offer you many other options and integrations with external apps and services. For example, you might choose to store your connection credentials to other tenants in secure storage like Azure Key Vault and fetch them in your Azure Pipelines to consume in deployment PowerShell scripts. Also, you can trigger deployment by some external service like Octopus Deploy or Jenkins. The final choice of the deployment method will depend on your existing infrastructure configuration and requirements for the desired solution.

And what is your experience with deploying to different tenants in Azure DevOps? Share it in the comments below 👇