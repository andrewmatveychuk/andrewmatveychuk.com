---
title: "How to update the OAuth2 permission grants for an Azure AD service principal"
seoDescription: "Learn how to update OAuth2 permission grants for Azure AD service principals using Microsoft Graph REST API"
datePublished: Mon Jan 24 2022 12:35:28 GMT+0000 (Coordinated Universal Time)
cuid: cm5b8ael0000409kt3f02ctvi
slug: how-to-update-the-oauth2-permission-grants-for-an-azure-ad-service-principal
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924764/96f36f75-9592-4d6f-9541-852b5a678dd2.png
tags: azure, how-to, oauth2, entra-id

---

When you register a third-party application in your tenant, a [service principal](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/service-accounts-principal) is created in your Azure AD to represent it. That service principal is used to define the permissions granted to the application and the permissions to manage the app itself in the tenant.

As a common practice, an application vendor defines all the permissions the app requires to function correctly, and you grant consent for those permissions while registering the app in a tenant. In some cases, e.g., working in regulated industries or environments with high-security requirements, you might need to reduce the list of permissions requested during the app registration to specific ones only.

In this blog post, I will share some tips on how to update the OAuth2 permission grants for an existing application (a service principal, if you want to be technically precise) in your tenant.

> Please note that changing (revoking) OAuth2 permission grants for an app might impact its functionality. If you are to modify permissions to a third-party application, make sure to consult with its vendor first.

## Problem statement

Suppose you explore the options available to you to review application permissions on the Azure portal. In that case, you might notice that most of them allow only removing all OAuth2 permission grants at once. The sample PowerShell script provided by Microsoft uses [the AzureAD cmdlets that don’t have the option to modify the existing grants](https://docs.microsoft.com/en-us/powershell/module/azuread/?view=azureadps-2.0#oauth2):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922572/598e144e-a06d-4298-a15a-5edead239a02.png align="center")

The recently updated [Update-AzADServicePrincipal](https://docs.microsoft.com/en-us/powershell/module/az.resources/update-azadserviceprincipal)cmdlet uses Microsoft Graph API and doesn’t have sufficient documentation on constructing input parameters to make it work properly. The same lack of documentation affects the [Update-MgOauth2PermissionGrant](https://docs.microsoft.com/en-us/powershell/module/microsoft.graph.identity.signins/update-mgoauth2permissiongrant) cmdlet from the Microsoft Graph PowerShell SDK, making using it quite problematic, too. My attempts to use the ‘Microsoft.Graph’ cmdlets ended up with an error stating that the scope property defining the grants is read-only and cannot be updated.

Using the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/ad/app/permission?view=azure-cli-latest) for the task doesn’t help either, as it can create new grants but not modify the existing ones.

## Solution

After unsuccessful attempts to use the Azure Portal, Azure PowerShell, Microsoft Graph PowerShell SDK and Azure CLI to update the OAuth2 permission grants, the only option left was to work with the Microsoft Graph REST API, which provides [an endpoint for updating the delegated permission grants](https://docs.microsoft.com/en-us/graph/api/oauth2permissiongrant-update).

The resulting execution sequence to update the OAuth2 permission grants for an Azure AD service principal will be the following:

1. Go to [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer) and log in with the admin account for your tenant. Consent for the 'Directory.ReadWrite.All' permissions.
    
2. Search for the Client ID of the service principal you want to update the grants to:  
    `GET https://graph.microsoft.com/v1.0/servicePrincipals?$search="displayName:your_app_name"`
    

> Pay attention to the **ConsistencyLevel** header that must be set to ‘eventual.’

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923514/e7eec958-35ff-4733-9533-1191fe2e1e36.png align="center")

3. Search for the OAuth2 permission grants by the Client ID you identified:  
    `GET https://graph.microsoft.com/v1.0/oauth2PermissionGrants`
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924166/be9a1da0-a362-4149-ab70-6b4ff99d21fa.png align="center")

4. \[Optional\] Get the specific grant only  
    `GET https://graph.microsoft.com/v1.0/oauth2PermissionGrants/{id}`  
    Where ID is the unique identifier of the grant and NOT the Client ID of your app. It has a format different from the common GUID-like identifiers.
    
5. Update the required grant with a new scope by using the PATCH method:  
    `PATCH https://graph.microsoft.com/v1.0/oauth2PermissionGrants/{id}`  
    and providing a request body in the following format:  
    `{ "scope": " offline_access Directory.AccessAsUser.All Directory.Read.All Group.ReadWrite.All Group.Read.All GroupMember.Read.All GroupMember.ReadWrite.All User.Read.All User.Read" }`  
    The scope property of the request must enumerate the complete list of granted permissions separated by **spaces**.
    
6. \[Optional\] Verify that the OAuth2 permission grant was updated with the scope provided earlier:  
    `GET https://graph.microsoft.com/v1.0/oauth2PermissionGrants/{id}`
    

Additionally, you can check that the Azure portal displays the updated app permissions, too.

I hope that the workaround will be helpful for anyone looking to modify existing app permissions without completely revoking them or deleting the app registration.

If you want to get recent updates to my blog, please subscribe with your email using the form below 👇