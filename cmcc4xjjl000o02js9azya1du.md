---
title: "WordPress as a Radius application"
datePublished: Wed Jun 25 2025 15:54:35 GMT+0000 (Coordinated Universal Time)
cuid: cmcc4xjjl000o02js9azya1du
slug: wordpress-as-a-radius-application
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1750862355409/c2d18432-63b1-4265-84d0-9d710e8eebed.png
tags: wordpress, radius-platform, radapp

---

I’ve been watching the development of [Radius](https://radapp.io/), a new application hosting platform that [originated in the Microsoft Azure Incubations team](https://azure.microsoft.com/en-us/blog/the-microsoft-azure-incubations-team-launches-radius-a-new-open-application-platform-for-the-cloud/) and later became a [CNCF project](https://www.cncf.io/projects/radius/), for a while, and I finally managed to dedicate some time to trying it in practice.

Radius is still early in development, but it can be an interesting option for exploring [platform engineering](https://learn.microsoft.com/en-us/platform-engineering/what-is-platform-engineering) practices. One way to see how promising it might be is to try deploying a simple application on it. In this post, I will try to show you how to run WordPress locally as a Radius application.

> I recommend visiting the [Radius official website](https://docs.radapp.io/concepts/why-radius/introduction/) to learn more about how it differs from Kubernetes and what challenges it can help solve. It has plenty of guides and tutorials to help you get started with the Radius platform and run your applications on it.

# Installing Radius locally

First, we must [install and configure the required Radius components](https://docs.radapp.io/installation/): the Radius command line (rad CLI) and the Radius control plane. While the **rad CLI** can be installed locally on your machine, the platform control plane runs as a Kubernetes application. The initialization process for the Radius control plane using rad CLI is pretty straightforward, and you can use your preferred Kubernetes setup either locally or as a cloud service.

After the basic setup is finished, you can check your Radius installation status via the command line:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862372358/bca59dea-99f8-4276-8c08-3e64586561cc.png align="center")

Later, when you deploy your Radius applications, you can also use the [Radius Dashboard](https://docs.radapp.io/guides/tooling/dashboard/overview/) to check for all system properties:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862412595/684c1fc8-1187-434b-91b2-92e3ed8bf9ac.png align="center")

Now, let’s take a step back for a moment and check how applications are defined in Radius.

# Understanding Radius primitives

When you deployed the Radius control plane using the rad command line, you essentially initialized your first Radius environment (the *default* one). A [Radius Environment](https://docs.radapp.io/guides/deploy-apps/environments/overview/) is an abstraction of your infrastructure to run your Radius-defined applications using predefined configurations and infrastructure components defined via [Radius Recipes](https://docs.radapp.io/guides/recipes/overview/). Your Radius environments can include [cloud providers](https://docs.radapp.io/guides/operations/providers/) like Microsoft Azure and Amazon Web Services, as well as [external identity providers](https://docs.radapp.io/guides/deploy-apps/environments/overview/#external-identity-provider) (Microsoft Entra Workload ID only for now).

You can combine multiple Radius environments into [Radius Workspaces](https://docs.radapp.io/guides/operations/workspaces/overview/), which are defined locally on the client side, to switch between different application contexts easily.

Radius Recipes abstract the configuration of infrastructure components in specific Radius environments. Your [Radius Application](https://docs.radapp.io/guides/author-apps/application/overview/) uses Recipes and defines dependencies and relationships between your application components.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862434555/53693a48-68d8-4590-ade5-94c78a159258.png align="center")

It might take you some time to grasp those Radius concepts, and it might be easier to do so while defining and deploying our demo application with Radius.

# Defining a Radius application for WordPress

With Radius, you can [define and run your applications](https://docs.radapp.io/tutorials/new-app/) using Azure Bicep, Terraform, and even Helm charts. For the sake of this demo, I will define our WordPress application using Bicep.

%[https://gist.github.com/andrewmatveychuk/0dc142bb885c6acb03eb266d1d4e8589] 

Here, we are deploying WordPress as a container that will run locally in our default environment as a Kubernetes service. However, that won’t be enough to have a functional WordPress website, as [it requires a database to run](https://wordpress.org/about/requirements/).

Now, let’s add a backend database here. To do so, we can [create a custom Bicep Recipe](https://docs.radapp.io/guides/author-apps/portable-resources/howto-author-portable-resources/) to provide our Radius application with an infrastructure abstraction to use. In my case, I chose to create a Radius recipe for a MySQL database:

%[https://gist.github.com/andrewmatveychuk/93d714be53ec9e625364e0e64e227fd5] 

Looking at that code, you might notice that creating custom Radius recipes can be a bit challenging. Usually, a platform team abstracts the infrastructure management complexity from developers.

As that recipe is intended for local development, the MySQL service also runs as a container. Plus, we are exposing it as a Kubernetes service so that our WordPress resource can connect to it.

After you import that recipe into your environment, you should be able to see it in the list of available recipes to use:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862458342/575f3e40-53cf-448f-a860-4b1601933df8.png align="center")

Now, we can update our application definition to leverage that local recipe:

%[https://gist.github.com/andrewmatveychuk/3340bd04b82fb3339a7d0367f27b8074] 

If we put ourselves in the developer's shoes, we can see that provisioning the database resource is quite easy, as all the nuances of its configuration are encapsulated in the corresponding Radius recipe.

> For the complete code of this project, please feel free to check my [**radius.demo** project on GitHub](https://github.com/andrewmatveychuk/radius.demo).

# Checking how it works

Let’s run our updated application definition and check the database configuration in WordPress:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862475966/b6eba065-a9fd-4c22-8347-f4369a536992.png align="center")

As you can see, it refers to our MySQL server using the internal Kubernetes name service created as part of the MySQL resource deployment with the recipe we created.

The [Radius Application Graph](https://docs.radapp.io/guides/author-apps/application/overview/#query-and-understand-your-application-with-the-radius-application-graph) will also show us the application layout with the MySQL resource dependency:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862529830/29ae0923-0b09-4718-84b7-0eff9f145d93.png align="center")

So good, so far. Our simple WordPress application is up and running on Radius.

Now, let’s look at how our resources are represented at the Radius control plane:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1750862548999/024cb058-9a5f-46c1-97cd-dd58a01362d3.png align="center")

As you might have already noticed, we provide the database connection properties for the WordPress container as environment variables. The downside here is that our database credentials are exposed in plain text, which is not ideal. Unfortunately, the secret management on the Radius platform is far from being production-ready. The documentation is limited to [using secret stores for certificate management](https://docs.radapp.io/guides/author-apps/secrets/overview/) and to [working with Azure Key Vault](https://docs.radapp.io/guides/author-apps/containers/volume-keyvault/). [Passing the secrets as parameters to resources](https://github.com/radius-project/radius/issues/8220) and injecting them back into your application without exposing them is still problematic.

Apart from that, our basic WordPress setup in the mentioned configuration is fully ephemeral, meaning we don’t have any persistent storage for our data, such as the MySQL database and website content files.

# Next steps

Now, you should have a basic understanding of a typical app configuration, consisting of a frontend web service and a backend database running in your local environment using Radius. It might not look like a good time investment, especially considering the management overhead of running such a simple app with a Radius dependency on Kubernetes and defining all required Radius abstractions. However, in the next post, I will explore how to deploy the exact application definition to Azure cloud native services using Radius Environment abstraction, which allows you to have a completely different infrastructure setup from the one we used locally.

We will also check how to add a state to your application by using persistent storage.

Do you want to know more about using Radius to host your applications? Let me know your thoughts in the comments below 👇