---
title: "How to connect to Azure Database for MySQL from Ghost container"
seoDescription: "Learn to securely connect Ghost on Azure to an Azure Database for MySQL using a Docker container and Bicep deployment"
datePublished: Mon Sep 06 2021 15:39:20 GMT+0000 (Coordinated Universal Time)
cuid: cm5dng2zt000d09l3dvj926fx
slug: how-to-connect-to-azure-database-for-mysql-from-ghost-container
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923694/b385b955-9cc3-45a5-b4ae-afb2dc36fbcf.png
tags: azure, mysql, ghost, containers

---

Continuing [the topic of hosting Ghost on Azure](https://andrewmatveychuk.com/tag/ghost-tag/), I decided to document some nuances of connecting to [Azure Database for MySQL](https://docs.microsoft.com/en-us/azure/mysql/) from a Ghost Docker container hosted on [Azure Web Apps for Containers](https://docs.microsoft.com/en-us/azure/app-service/configure-custom-container). Moreover, I will shed some light on doing it securely from any Node.js application in this post.

If you are more interested in the technical details, feel free to skip my reasoning for using Azure Database for MySQL as a backend for Ghost in the first section and scroll straight to the second one.

## Why fall back to a managed MySQL server from the multi-container app

In Dec 2020, when I came up with the idea of [hosting Ghost on Azure as a multi-container app using Docker Compose support in Azure App Service](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers), that configuration worked fine. I reduced the application hosting costs by hosting MySQL as a container on the same App Service plan as the Ghost container. However, starting from May 2021, I received [a few notifications indicating that the original deployment configuration stopped working](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers/issues?q=is%3Aissue+is%3Aclosed), and the following error occurred in App Service container logs:

*ER\_HOST\_NOT\_PRIVILEGED: Host ‘172.X.X.X’ is not allowed to connect to this MySQL server*

Nothing was changed in the deployment configuration, and I started looking for possible root causes of that issue.

As [the multi-container support on App Service is still in preview](https://docs.microsoft.com/en-us/azure/app-service/configure-custom-container?pivots=container-linux#preview-limitations), changes to that service might have caused the containers to crash. Unfortunately, I didn’t find any mentions of service updates in the official docs or on community resources.

Another possible cause of the error is some updates to the MySQL container image — I was using the image with the ‘mysql:5.7’ tag and didn’t stick to a specific build in that minor version.

The attempts to redeploy the app using a specific MySQL container image that was up to date when I created the original Docker Compose configuration didn’t succeed. Neither successful were attempts to upgrade the setup to using a MySQL 8 container. Also, it is worth mentioning that the same configurations that produced errors while running on App Service worked like a charm in a local Docker Desktop environment, so the error wasn’t related to [the well-known MySQL permission nuance when a user account can connect to the server from localhost only](https://dev.mysql.com/doc/refman/8.0/en/docker-mysql-more-topics.html#docker_var_mysql-root-host).

After troubleshooting the issue for a few days, to say that I was confused was to say nothing. Continuing to play a game of numbers when you don’t even know the details of how Docker networking works in App Service didn’t seem reasonable. So, it was high time to migrate to something more stable, supported and well-documented.

## Setting up Azure Database for MySQL

You can definitely go to the Azure portal and use the provisioning wizard to create a new instance of the Azure Database for MySQL. However, that approach didn’t fit [my idea of an unattended, reproducible, easy-to-use one-click deployment concept](https://andrewmatveychuk.com/a-one-click-ghost-deployment-on-azure-web-app-for-containers). As I already implemented that concept with ARM templates and [the Deploy to Azure button](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-to-azure-button), it was natural to refactor the existing deployment templates to use a managed MySQL database instead of the MySQL container. Apart from that, as [Azure Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) – a new declarative language for Azure resource deployment reached its general availability, I decided to migrate my deployment configuration to that new format completely.

> The Bicep team has already published many [Bicep snippets for Azure resources](https://github.com/Azure/bicep/tree/main/docs/examples), so you can use them as a reference.

After some trials and errors, my resulting configuration for Azure Database for MySQL resource looked like the following:

%[https://gist.github.com/andrewmatveychuk/658beb5b8e7e2ecf29e81c362a57fd8e] 

Leaving the Bicep syntax nuances aside, the resource configuration has a few important aspects to consider. First, it enforces [encryption for server connections](https://docs.microsoft.com/en-us/azure/mysql/howto-configure-ssl) and sets the minimal version of TLS for incoming connections. Secondly, it configures [server firewall rules](https://docs.microsoft.com/en-us/azure/mysql/concepts-firewall-rules) to accept incoming connections from Azure services only. Lastly, it uses password authentication.

The database password is specified during a template deployment and saved in an Azure Key Vault. The App Service hosting the Ghost application uses that password via [Azure Key vault secret reference](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references) to authenticate to the database.

Now, let’s look into the updated Ghost app configuration.

## Configuring Ghost app for Azure Database for MySQL

The Ghost team provided [comprehensive documentation on configuring a Ghost database connection to MySQL](https://ghost.org/docs/config/#database). That documentation even contains a detailed description of different client SSL options. To pass that nested JSON key structure as configuration for Azure App Service, [all semicolons should be replaced by \_\_ (double underscore)](https://docs.microsoft.com/en-us/azure/app-service/configure-common#add-or-edit) to represent the same configuration as key-value pairs in the app settings.

Apart from that, Ghost, as a Node.js application, also relies on the native runtime TLS options. That said, in the SSL section of your config, you can specify all [‘tls.createSecureContext’ options](https://nodejs.org/api/tls.html#tls_tls_createsecurecontext_options). For example, in addition to providing your custom certificate (if needed), you can set the minimum and maximum TLS versions to be used when establishing connections:

%[https://gist.github.com/andrewmatveychuk/f872c6b1cbde374c7a544fd60d72d0b8] 

> Pay attention that [the ‘secureProtocol’ configuration option widely referenced on the Internet](https://forum.ghost.org/t/configure-mysql-over-tls-ssl/2297/8) is **deprecated** and should not be used.

Please note that the Baltimore CyberTrust Root certificate is included in [the Mozilla Included CA Certificate List](https://ccadb-public.secure.force.com/mozilla/IncludedCACertificateReport). As documented in [the Azure Database for MySQL documentation (Configure SSL)](https://docs.microsoft.com/en-us/azure/mysql/howto-configure-ssl#step-1-obtain-ssl-certificate), you **don’t need** to download and embed it in your Ghost configuration.

## Try sample configuration

After much work on migrating from ARM templates to Bicep and configuring the Ghost app to use Azure Database for MySQL, the new one-click Ghost deployment on Azure App Service is finally ready. You can quickly try it out by clicking the Deploy to Azure button in [the project repository on GitHub](https://github.com/andrewmatveychuk/azure.ghost-web-app-for-containers/issues?q=is%3Aissue+is%3Aclosed).

As Bicep makes it much easier to build modular templates, I discontinued two separate templates for deployments with Azure CDN and Azure Front Door and created the single deployment templates with conditional deployment logic – you need to select the desired option:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922829/7233da1c-e651-440e-bf00-c8dde3566595.png align="left")

If you experience any issues with the deployment, please submit an issue on GitHub or post it in the comments below 👇

Stay tuned to hear about my impressions on the new authoring experience for Azure deployments using Azure Bicep 😉