---
title: "How to fix images width for Microsoft Outlook in Mailchimp RSS campaigns sourced from Ghost"
seoDescription: "Fix Outlook image sizing in Mailchimp RSS campaigns with Ghost by customizing RSS feeds and using Media RSS for proper image dimensions"
datePublished: Wed May 01 2019 17:15:00 GMT+0000 (Coordinated Universal Time)
cuid: cm5b7y2ep000009ladg79gcur
slug: how-to-fix-images-width-for-microsoft-outlook-in-mailchimp-rss-campaigns-sourced-from-ghost
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1737045856961/2b659d19-a4c0-495f-928d-d69c70fe3cf8.png
tags: blogging, ghost, how-to

---

Recently, I was playing with email notifications for my blog, which is powered by [Ghost](https://ghost.org/). I was choosing between [Drip](https://www.drip.com/) and [Mailchimp](https://mailchimp.com/) and decided to try them for the latter. It took me a few days to learn the basics of Mailchimp lists, campaigns, templates and automation features. I designed my first RSS campaign to share blog updates with my subscribers on a weekly basis. When everything was configured and ready, I sent a test email to myself and ran into the following problem:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681922859/bec5fa6f-03ce-45a2-a8df-99a804239709.jpeg align="center")

The post’s cover image, which should have been resized to the email template's fixed width, was served in its original size, causing the whole message to be blown up.

## Context

As it seems from a technical perspective, both Mailchimp and Ghost have all the required features to build an automated pipeline for sending out emails. Mailchimp service has a useful [campaign template that can be used to automatically email updates on the new posts from your blog to your subscribed contacts](https://mailchimp.com/help/share-your-blog-posts-with-mailchimp/). This template uses an RSS feed as a source and sends out emails by a defined schedule. Ghost engine has the automatically generated RSS feed for the main post index and the options for [RSS customization](https://ghost.org/integrations/custom-rss/).

Both parts were properly configured and ready to send a weekly digest of blog updates. Before putting the whole system into production, I decided to perform an end-to-end test to ensure that everything worked as expected. As part of the verification process, a test email was sent to my email address.

## The problem

Unfortunately, when I viewed my test email in my desktop Outlook client, the cover image was not resized to the email width and looked like on the screen at the beginning of this post. However, in the web version of Outlook and in Gmail, the same email was rendering correctly without being stretched by the post image.

## Investigation

Mailchimp has a feature that resizes the images from the RSS feed to fit the template. Trying to turn it off and on didn’t help. In fact, [this feature simply doesn’t work for emails viewed in the desktop version of Outlook](https://mailchimp.com/help/troubleshooting-rss-in-campaigns/).

After some searching and [playing with different custom CSS styles](https://www.google.com/search?ei=T43IXIaNN-3KrgS3j4LwBg&q=resize+rss+images+to+mailchimp+template+width&oq=resize+rss+images+to+mailchimp+template+width&gs_l=psy-ab.3...9314.13989..14970...0.0..0.339.881.1j0j2j1......0....1j2..gws-wiz.......0i71.ge2lrVWHhx8) in the email template with no positive results I came across [the explanation of such strange results on image rendering in Outlook client](https://mailchimp.com/help/my-campaign-looks-bad-in-outlook/). As it turned out, the old versions of Outlook clients used Internet Explorer for rendering HTML emails and the newer ones used Word engine to do this job. As a result, the [Outlook client just doesn’t recognize such CSS elements as max-width, max-height](https://www.campaignmonitor.com/css/email-client/outlook-2007-16/), and so on that are used for altering the image size in Mailchimp email templates.

The last option was to serve Mailchimp with images of the proper size. Reducing the size of the original images on the web pages was not an option as it would gradually reduce the user experience for the site’s visitors. I had to find a way to transform images from the site’s content to the Mailchimp campaign.

## The solution

As I already mentioned, [Ghost engine has the options for customizing RSS feeds](https://docs.ghost.org/integrations/custom-rss/) and they can be used on the site theme level. This means that you don’t have to change anything in the Ghost engine or its configuration itself. You can redefine your default RSS feed or, even better, create a new one without affecting the default just by editing your current theme.

RSS has a module extension – [Media RSS](http://www.rssboard.org/media-rss), which extends the capability to syndicate different media files in your feed. Moreover, its [*media:content*](media:content) element has attributes to define the height and width of the media object. So, for example, you can set the image size that will be served over RSS to a subscriber:

`<media:content url="{{feature_image}}" medium="image" height="auto" width="560"/>`

I created a new template for the RSS feed to be used as a source for the Mailchimp campaign and configured its URL in routes configuration. After this, I changed the **RSS feed URL** in my campaign to [this new address](https://andrewmatveychuk.com/mailchimp/rss/), and, voilà, my new test email was rendering in Outlook client correctly:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734681923864/abd479d6-fa12-4e23-be0d-e0dbbdbe8276.jpeg align="center")

P.S. Remember also to add to your Mailchimp email template [a CSS style for correct image rendering on mobile devices and other responsive clients](https://woorkup.com/fix-max-width-images-in-mailchimp-rss-to-email-campaign/) so it looks something like this:

`\*|RSSITEM:IMAGE|\*`