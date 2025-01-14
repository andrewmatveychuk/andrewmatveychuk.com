---
title: "Migrating from Ghost to Hashnode"
seoTitle: "Switching from Ghost to Hashnode"
datePublished: Fri Dec 20 2024 09:01:00 GMT+0000 (Coordinated Universal Time)
cuid: cm4witd9d000k09mqhknpbg1x
slug: migrating-from-ghost-to-hashnode
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1736865300011/eb0feaf1-acbe-44db-934c-857a96834641.png
tags: ghost, migration, hashnode

---

## Announcement

I’ve migrated my blog from [Ghost(Pro)](https://andrewmatveychuk.com/moving-to-ghost-pro) to [Hashnode](https://hashnode.com/). All links should be working, but please let me know if you encounter something broken or missing.

## Reasons

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736864510883/b4c30f86-56eb-4681-8775-ae18c77fd93e.png align="center")

[I used Ghost to run my blog for many years](https://andrewmatveychuk.com/tag/ghost), self-hosting it first and [becoming a paying customer of Ghost(Pro)](https://andrewmatveychuk.com/moving-to-ghost-pro) later. I would say that I was mostly satisfied with it as a blogging platform. Nevertheless, I decided to try something new last year, and Hashnode was on my radar for a while.

Ghost(Pro) has its niche in various modern blogging platforms. It’s feature-reach, reliable, SEO-optimized, and fast. However, the overall direction of its evolution and its pricing model push you toward monetizing your blog as your audience grows. Besides, its [**Stater** pricing tier](https://ghost.org/pricing/) doesn’t allow you to use custom themes, significantly reducing your ability to customize your blog. You need to go with the **Creator** tier for that, which increases your annual spending on blog hosting three-fold. Those aspects made me question if there are better alternatives for the Ghost(Pro) Stater tier.

I think Hashnode, being relatively younger, cannot be compared to Ghost head-to-head. Also, it has a different product placement, positioning itself as [a community blogging solution](https://hashnode.com/feed), which is [somewhat similar to Medium](https://townhall.hashnode.com/hashnode-is-the-new-medium-for-the-tech-community). At the same time, you can map a custom domain to your Hashnode blog or self-host using the [Hashnode Headless CMS](https://docs.hashnode.com/blogs/getting-started/hashnode-headless-cms), which allows you not to be locked into using Hashnode as a managed platform only. Basically, you can have almost the same platform functionality on Hashnode for free as you could have it on the Ghost(Pro) Stater tier if you don’t need to use a paywall for your subscribers.

Apart from that, Hashnode allows you to import and export [(backup) your articles to a GitHub repository](https://docs.hashnode.com/blogs/blog-dashboard/github/how-to-set-up-automatic-github-backups-on-your-blog) in the Markdown format. Even though I have the source documents for my articles and the related images stored in my OneDrive folder and backed up on the file level, manually restoring them would be a tedious task, considering the growing number of blog posts. Plus, I tend to edit many blog posts on the website after publishing them, like adding updates and fixing broken links, which makes my source Word documents somewhat outdated. Having all my blog posts up-to-date, versioned, in the Markdown format, and automatically syncing to my GitHub repository as a backup location makes it much easier to restore or transfer to another blogging platform if need be. Compared to Ghost, where your blog posts are stored in a database and can only be exported in a cumbersome JSON format on-demand, that GitHub integration was the most convincing reason to switch to Hashnode. Because, you know, having backups is what distinguishes good engineers from not-so-good ones.

## Migration process

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736864524761/741f8e2d-208b-4efe-aa65-2fc9cb38bf23.png align="center")

Unfortunately, there is very little information on the Internet about migrating your content from Ghost to Hashnode. Some people migrated their content [programmatically](https://thao.pw/simplifying-blog-migration-with-automation-ghost-to-hashnode), while others leveraged [the Hahnode feature to import content from RSS feeds](https://itheo.tech/import-ghost-cms-articles-into-hashnode), pointing it to their current Ghost-based blogs.

I wanted to spend as little time as possible on the migration and chose the migration via [the RSS import](https://docs.hashnode.com/blogs/blog-dashboard/import/rss-importer). Unfortunately, when I was ready to start the import process, that feature was turned off for unknown reasons. It was time to speak to Hashnode Support, and I was pleasantly surprised by their responsiveness and the workaround they provided.

It turned out that the Hashnode team (kudos to [Favourite Jome](https://hashnode.com/@Favourite)) had already created [a migration app to migrate your content from Ghost to Hashnode](https://ghost-hashnode-migration.vercel.app/). You only need to provide it with the JSON export of your Ghost blog and the API token for your new Hashnode blog, and it will import most of your existing blog content, saving most of the source formatting and media, including images, GitHub gists, and other embeds. Your posts will be imported as drafts in Hashnode, so you can (and should) review them before republishing. Depending on the formatting of your original posts and the different embeds you might use, you might need to spend some time fixing some import errors, formatting, and checking for missing content.

In my case, it took me a couple of days to properly format my drafts on Hashnode, check that all images were in place and displayed correctly, and fix a few dozen broken links in some of my old posts.

Apart from importing your old posts from Ghost, you might also need to import your subscribers. [The subscriber export in Ghost](https://ghost.org/help/exports/#members) and [their import in Hashnode](https://docs.hashnode.com/help-center/hashnode-newsletter/importing-your-subscriber-list-into-hashnode-newsletter)v use the CSV file format for data, so migrating them wasn’t a big deal. It took me just a few PowerShell commands to clean up unnecessary fields before the import, and I was all set to send out mail notifications to my subscribers from my Hashnode blog.

Currently, [managed Hashnode hosting doesn’t support custom themes](https://docs.hashnode.com/blogs/blog-dashboard/appearance), and your customization options are limited to three predefined layouts, custom logos, and a branding color. However, it wasn’t an issue for me, as the resulting blog layout is very similar to my previous one in Ghost.

A few final touches were related to remapping my custom domain, transferring my custom page redirects, and repointing my user analytics projects to the new blog. Those hardly require mentioning, as their configuration was pretty straightforward and well-documented. I particularly liked that on Hashnode, you don’t need to mess up with code injections for Google Analytics or other similar solutions to make them work, and you only need to provide your unique tracking tags or IDs for configuration.

## Back to blogging

Now, I’m finally set up to continue my blogging on Hashnode, and I shall see what my impression of it is after using that platform for a while. I’ve already stopped drafting my articles in Microsoft Word and completely switched to the Hashnode editor. The ability to seamlessly switch between the WYSIWYG and raw Markdown editor modes is fantastic, and I’m using it to learn more about Markdown formatting. Plus, having my articles automatically backed up to a GitHub repo makes me less worried about restoring them if needed.

Although the current blog can run on Hashnode with the custom domain entirely for free, I happily paid for the [Hashnode Pro](https://townhall.hashnode.com/meet-hashnode-pro) to support the team behind the great product. I hope that the Hashnode team will continue releasing new features and extending the functionality of the existing ones.

P.S. Please treat all of the above as my migration experience, which I’m happy with, and not a Hashnode advertisement in any way. If you have questions about my migration to Hashnode, please feel free to ask them in the comments 👇