---
title: "How I failed the network configuration for my blog"
datePublished: Tue Jun 16 2020 12:38:57 GMT+0000 (Coordinated Universal Time)
cuid: cm5b8hfcw000709l52tyr35zx
slug: how-i-failed-the-network-configuration-for-my-blog
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737043944702/c4b1d686-1c97-4c07-9215-ab198eb7bff1.png
tags: blogging, dnssec

---

## Prelude

A few months ago, I registered an additional shorter domain name for my blog – [matveychuk.com](https://matveychuk.com/) and was tweaking the DNS configuration so that users could access the site using both domains. My DNS provider supports [the URL record type](https://support.dnsimple.com/articles/url-record/), from the technical perspective, that was a piece of cake. I just created the redirect records for the bare domain name and www sub-domain and was ready to go:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922787/c6c35691-656e-4b20-9b4c-a298f88d00df.png align="center")

As I didn’t expect it to be so easy and planned to spend more time on it, I decided to have a sneak peek at [DNSSEC](https://support.dnsimple.com/articles/dnssec/), an extension to regular DNS that allows a DNS client to verify that the query results weren’t tampered with in transit.

> Of course, there is [a complex process for ensuring the authenticity of a DNS record](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/) behind the curtains, but for the sake of this post, it is not very relevant.

Both [DNSimple and Cloudflare automated the configuration of DNSSEC](https://support.dnsimple.com/articles/cloudflare-ds-record/) for a domain and made it quite easy to set up, even for a not-very-technically proficient customer. So, after half an hour, the DS record was in place, and the Cloudflare control panel indicated that DNSSEC was successfully configured. The task is done, closed, and forgotten.

## First signs of trouble

In the meanwhile, my superior (Hi Dan! 😉) reached me regarding my site unavailability. That was quite unusual because I hadn’t heard any such complaints up to that moment. A quick check confirmed that the site is up and running ‘on my PC’ and can also be accessed by external services like [Pingdom](https://tools.pingdom.com/). I assumed that the issue might be temporary and specific to a particular ISP provider in Norway that my colleague used at his home office. As a confirmation of my assumption, the affected colleague was able to reach my site when connected to the corporate network via a VPN. From the IT Ops point of view, the workaround was found, the incident was resolved, and the user could use the service again.

## The problem continues

In a month or so after the first issue occurrence, I started receiving similar complaints from some of my readers. One person from New Zealand reported that my site seems to be down. Once again, a set of quick tests indicated that the site is running and reachable via both my landline and cellular providers. It seemed like yet another network fluctuation.

In another case, the reader who couldn’t access my site was located in NJ, USA. That time, I was able to get some client diagnostic information, which showed that the client computer couldn’t resolve my site’s DNS name. At the same time, the reader could successfully reach other services on the Internet.

That started really worrying me. I decided to go the extra mile and [set up ongoing availability monitoring and instrument the site with Azure Application Insights](https://andrewmatveychuk.com/how-i-run-my-blog). Still, a few days after that, I had no leads to the problem’s root cause.

## The moment of truth

Last week, my teammate, Jon, notified me about the issue with all the same symptoms I had observed before. After checking his client's DNS configuration, I noticed that he used [Google Public DNS](https://developers.google.com/speed/public-dns) on the home router. A lookup for [andrewmatveychuk.com](https://andrewmatveychuk.com) using Google DNS server IPs returned a generic non-existing domain error. Knowing how many system administrators, as well as home users, set up Google Public DNS for their name resolution, the scale of the problem made my palms sweat. I was also confused about why my blog’s domain name was resolved successfully by my ISP name servers and services like Pingdom and Azure but not by Google DNS.

A 5-minute search on the Internet pointed me in the direction of checking the DNSSEC configuration for my domain. Test queries on [https://dns.google.com/](https://dns.google.com/) confirmed that the domain name [resolved successfully without DNSSEC validation](https://dns.google.com/query?name=andrewmatveychuk.com&type=A&dnssec=false) and failed when using it:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923585/be6bafe7-929a-437d-89a4-9ce2ab732868.png align="center")

[The DNSSEC check with specialized tools](https://dnssec-analyzer.verisignlabs.com/andrewmatveychuk.com) confirmed that my configuration malfunctioned, breaking the chain of trust. In other words, all DNS servers that handle DNSSEC appropriately distrusted my domain name and were ‘not able’ to resolve it. This meant that I was seriously busted.

As it was almost impossible to figure out what I did wrong configuring DNSSEC more than a few months ago, I deleted the existing configuration and verified that the Google DNS stated resolving my domain name. After that, I went through the process of setting up DNSSEC once more, but this time, I ran all the checks to ensure that it was correct and actually worked:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681924201/771dde07-f27d-40e8-9b18-f7fb2a5cad93.png align="center")

## The consequences

Considering that the content on my blog was regularly crawled by the Google search engine and the new posts were indexed according to Google Search Console and the search results, I might assume that the described issue didn’t affect the search ranking. However, from the end-user point of view, for a significant portion of my readers, the experience of not getting the information they were looking for was quite unpleasant. If correlating the issue life span with the decrease of active users in Google Analytics over the same period, it might have affected almost one-third of my audience, which is a lot.

Could that issue be prevented? That’s a good question. From the technical perspective, that issue reminded me of the importance of having end-user experience monitoring for your services. Even if all ‘server-side’ metrics and logs seem fine, the consumers might still be affected by some weird bug or configuration issue, as in my example.

I’m considering options for improving that area of my blog. So, stay tuned and subscribe to the latest updates on my blog 👇 to follow up on this postmortem update!