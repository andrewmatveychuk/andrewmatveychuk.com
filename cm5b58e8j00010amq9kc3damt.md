---
title: "How I run my blog"
seoDescription: "Discover how to efficiently run a blog for under $8/month with fast loading times, encryption, and subscriber updates"
datePublished: Tue Jun 02 2020 03:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b58e8j00010amq9kc3damt
slug: how-i-run-my-blog
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737041839029/25e08b45-cfa3-4f92-b0eb-0b1156fc6dcf.png
tags: blogging, ghost, howtos

---

Recently, my friends and colleagues asked me a few questions about my blog, specifically about its internals. To avoid repeating myself and answering the same questions over and over, I am going to share the technical details here.

Also, to whet your appetites, *I spend less than $8 monthly to keep it running*. For that ridiculously low price, I have the pages loaded in less than 1 second, serve the traffic over encrypted HTTPS connections, monitor my site availability from multiple locations, and send out site updates via email to my subscribers.

So, let’s look under the hood.

## The engine

When I was considering multiple CMS options for running the blog, I looked for a few particular things. Firstly, as I don’t consider myself an experienced web programmer, it was essential that I could install the engine easily without digging into its source code. Secondly, I wanted it to be as simple as possible – it’s just a blog, man 😉 So, such heavyweight and multifunctional solutions as [Umbraco](https://umbraco.com/) or [WordPress](https://wordpress.com/) were out of scope.

[Ghost](https://ghost.org/) was an emerging solution at that time, and I decided to give it a try to serve the content of my blog. This engine attracted my attention because it is free, open-source, lightweight, highly customizable and has lots of external integrations. I also liked the option of either running it in a self-hosted environment or using it as a managed service.

To be honest, after a year of self-hosting (more on that later), I don’t consider Ghost to be a perfect solution for me. On the one hand, it does its job well and works fast. On the other hand, I really don’t like [its vendor lock for supported environments](https://ghost.org/docs/install/ubuntu/), and I would like to have more flexibility with OS and database options. Besides, from the IT Ops point of view, the breaking changes both in [Ghost](https://ghost.org/faq/upgrades/) and [Nodejs](https://nodejs.org/en/blog/release/v12.0.0/) platforms are a [pain in the ass](https://github.com/TryGhost/Ghost/issues/11855). So, for me, it is still a tradeoff between the engine’s functionality and its maintenance cost.

## The hosting

As I didn’t expect to have significant traffic just from the start, I chose to be lean and self-host the blog as [the pricing for Ghost managed service](https://ghost.org/pricing/) is a little bit overhead compared, for instance, with [the WordPress managed hosting](https://wordpress.com/pricing/). When the site reaches a considerable number of page views, the managed Ghost hosting will be an excellent choice to offload the infrastructure management.

From the list of suggested self-hosting options, I ended up with [the Ghost Droplet by DigitalOcean](https://marketplace.digitalocean.com/apps/ghost), which is a Linux-based virtual machine. So, technically, I have full control over the operational environment. [The pricing](https://www.digitalocean.com/pricing/) is extremely cheap, and for my purpose, the smallest droplet works just fine:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922555/99f99764-3973-48fb-a1b2-f3ba64a6a220.png align="center")

It costs me $6 per month to host my blog: $5 for the droplet, plus $1 for automated weekly backups.

## The design

I didn’t bother with the site design too much and used a slightly customized [Casper theme](https://github.com/TryGhost/Casper) – a default theme developed by the Ghost team. Despite having some issues with post cover image scaling, which I’m currently looking into, it serves my purpose as of now and works relatively well on desktop and mobile devices. In the upcoming year, I will probably change the design and switch to [something new](https://themeforest.net/search?platform=Ghost%203.x.x&sort=sales#content).

## The domain name

I have been hosting all my registered domain names with [DNSimple](https://dnsimple.com/) for years, and I’m pretty satisfied with their service quality and a wide range of technical features. Strictly speaking, DNSimple is [a DNS hosting service](https://support.dnsimple.com/articles/dnsimple-services/), and the domains are registered by [eNom](https://www.enom.com/), but that is more of a technical nuance.

The domain name for my blog costs me $17 per year: $14 for the domain itself, plus a $3 fee for [the WHOIS Privacy Protection](https://support.dnsimple.com/articles/what-is-whois-privacy/).

## The CDN

As you might have already noticed from the performance metrics of my hosting, the utilization of public network bandwidth is relatively low compared to more than 1,000 active monthly users on my blog. Of course, 1k is not a huge audience. Still, fast page loading is an essential metric from the usability and search engine optimization points of view.

[My average page load time](https://tools.pingdom.com/#5c969a8454000000) is less than 1 second 🚀 and that is possible because of [the Cloudflare CDN](https://www.cloudflare.com/cdn/). They have [a free plan for personal websites](https://www.cloudflare.com/plans/#free-modal) that provides you with worldwide CDN, network threat protection, and more.

Thanks to the CDN, almost 98% of the requests don’t even reach the server running the site, and I can save money on the hosting as I simply don’t need to use a more powerful and, therefore, more expensive server:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923490/37992e77-200d-483d-ab19-8bdfac0708d1.png align="center")

Sure, there is still room for improvement. For example, I am considering migrating from the Facebook comment plugin to Discuss, as the Facebook script is quite heavy and cumbersome.

## The security

Apart from offloading most of the traffic from my hosting, Cloudflare allows me to use SSL encryption for my blog and do it literally for free. The service automatically assigns a certificate to your Cloudflare logical endpoint, plus you can also configure encryption between Cloudflare and your origin server:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924104/fc85bef4-eb83-43d1-841a-75802774ae6e.png align="center")

Apart from the site security and speed improvements, that configuration also ultimately diminishes the need for certificate provisioning, configuration, and periodic rotation. For people who are familiar with the tedious annual process of updating certificates, it seems like a blessing 🙏

## The analytics

I use Google Analytics for usability and end-user monitoring. As I’m not super focused on search optimization and user acquisition for my blog, I look into GA primary for some basic stats like the number of active users and the most visited web pages.

Recently, I have also instrumented the site with Application Insights to have better visibility over its internals and to monitor its availability from multiple locations. It adds approximately 50 cents to my monthly bill but provides me with valuable diagnostic information and notifies me if there is something wrong with the site:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924761/94575015-b07b-48ac-a487-c772b85c34a3.png align="center")

Also, it would be interesting for me to compare the analytical data from Google Analytics and Application Insights side-by-side in a while.

## The subscription

To keep my readers updated, I provide them with an option to [subscribe to the recent updates to my blog](https://andrewmatveychuk.com/#subscribe). When I started, Ghost introduced [the Subscribers feature](https://ghost.org/docs/api/v2/handlebars-themes/subscribers/) that allowed me to collect user emails and feed them to external email subscription services like [MailChimp](https://mailchimp.com/) or [Drip](https://www.drip.com/). After some consideration, I stuck with MailChimp due to [its free plan for up to 2,000 subscribers](https://mailchimp.com/pricing/), as I simply didn’t need most of the advanced features from the paid plans.

Also, to automate the subscription process, I used [Zapier](https://zapier.com/) to monitor my Ghost instance for new subscribers and add them to my MailChimp list:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925394/352f5c15-e7f1-4838-a957-41770d3b4f27.png align="center")

In MailChimp, I configured an ongoing weekly campaign based on my blog RSS feed. The campaign sends out all updates, if there are any since the last campaign run:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681925976/53a7c11a-e11a-4224-b510-5e48dcf8c161.png align="center")

[The subscription configuration probably took me the most time during the initial setup](https://andrewmatveychuk.com/how-to-fix-images-width-for-microsoft-outlook-in-mailchimp-rss-campaigns-sourced-from-ghost) as it involved connecting multiple services. Luckily, now the whole process is fully automated – I just need to publish new posts on the site to have them delivered to my subscribers.

So, if you too want to be informed about my last blog posts, hit the subscribe button below and opt-in for your subscription in the follow-up email 👇

See you soon!