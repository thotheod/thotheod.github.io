---
layout: post
title:  "Add your custom domain to your GitHub Pages blog"
date:   2023-11-06 17:38:00 +0300
categories: blog tech
tags: blog github-pages azure dns custom-domain
---

This post marks the conclusion of our 'informal trilogy' on how to create a blog for free. In the first part, we explored [how create it with jekyll](/posts/how-I-created-this-blog/) and how to host it on GitHub Pages without any cost. The second part detailed the process of obtaining a [free custom domain through your Azure Subscription](/posts/custom-domain-azure/).

In this post, we will focus on utilizing that custom domain in your newly created blog. This way, you can personalize your URL to something other than the default `<username>.github.io`, allowing you to choose a web address that truly suits your style.

## Supported Custom Domain Types
GitHub Pages [support both subdomains and apex domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages#supported-custom-domains), but in this part of the series we will describe how you can use a *custom subdomain*. The reason for that is if you use a custom subdomain, you still have the flexibility to use more custom subdomains for other projects or tests. For example, in my case a created the custom domain `nimbus-musings.com`. We will describe below the process to create a custom subdomain named `blog.nimbus-musings.com` and use it for our blog. In that case, I can still use more custom subdomains in the future for other testing purposes, like creating some named as sampleapp01.nimbus-musings.com, sampleapp02.nimbus-musings.com, and so on. 

## Configure Custom Subdomain
1. Go to GitHub and navigate to your site's repository; that should have a name following this pattern `<username>.github.io`. 
2. Go to [your repository's settings](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain) and under *code and automation > Pages*, add the custom subdomain (i.e. *blog.example.com*) as shown below.

![Add Custom subdomain in GitHub Pages repository](/images/custom-domain-gh-pages/01-gh-add_custom-domain.jpg){: width="400" height="200" }
_Add Custom subdomain in GitHub Pages repository_

3. The DNS check will fail because your custom subdomain is not yet connected to your blog.  You need to create [CNAME Record](https://en.wikipedia.org/wiki/CNAME_record) in your Azure DNS zone that will map your custom subdomain (alias) to the canonical name (i.e. *<username>.github.io*). 
4. Navigate to your Azure Portal, select the correct subscription and resource group, and find your DNS Zone. The Azure DNS zone resource is named by the custom domain that supports (i.e. example.com). When you select the DNS zone, click on the *+ Record Set* and create a new CNAME record as shown below.

![Add CNAME](/images/custom-domain-gh-pages/02-Add%20record%20set.jpg){: width="400" height="200" }
_Add CNAME to your username.github.io canonical name_

5. If you go back to your [repository's settings](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain) and then under *code and automation > Pages* you  will still see that the DNS check is either in progress or has failed. As long as this DNS check is not successful, you cannot "Enforce HTTPS," which is highly recommended. 

![DNS Check in progress](/images/custom-domain-gh-pages/04-dns_check_in_progress.jpg){: width="400" height="150" }
_DNS Check in progress_

### Verify Your Custom Domain
It is essential to [verify your custom subdomain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages#verifying-a-domain-for-your-user-site) for GitHub Pages. Once you successfully verify your custom subdomain, you essentially prevent other GitHub users from taking over your custom domain and using it to publish their GitHub Pages site. Therefore, it is strongly recommended that you do not use wildcard DNS records, such as `*.example.com`. These records put you at an immediate risk of domain takeovers, even if you verify the domain. To verify your custom subdomain: 
1. Click your profile photo, then click **Settings**
2. Click on **Pages** (under *Code, planning, and automation*)
3. Click on **Add a domain** and enter your exact subdomain, i.e. `blog.example.com` (and not `*.example.com`). Once you press the "*add domain*" button you will be presented with similar information as shown below:

![GitHub Pages TXT code for domain verification](/images/custom-domain-gh-pages/05-Add%20a%20Pages%20verified%20domain.jpg){: width="400" height="200" }
_GitHub Pages TXT code for domain verification_

4. Go to your azure portal, find your Azure DNS zone resource and click on the *+ Record Set* to create a new [TXT record](https://en.wikipedia.org/wiki/TXT_record) as shown below

![Create a TXT Record](/images/custom-domain-gh-pages/06-Add%20txt%20record%20set.jpg){: width="210" height="200" }
_Create a TXT Record_

5. Once you create the TXT record, back on your GitHub Pages settings the domain you have added will be tagged with a green "verified" tag

![GitHub Pages Verified Domain](/images/custom-domain-gh-pages/verified%20domain.jpg){: width="400" height="150" }
_GitHub Pages Verified Domain_

Now that your custom domain is configured and correctly verified, you can [enable HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https#enforcing-https-for-your-github-pages-site), which adds a layer of encryption that prevents others from snooping on or tampering with traffic to your site.

## Final Thoughts

In this concluding chapter of our three-part series on creating and hosting a blog for free, you've learned how to customize your blog's web address to something that truly represents your unique style. By configuring a custom subdomain on GitHub Pages and verifying it, you've not only personalized your URL but also secured it against potential takeovers. With your custom domain correctly configured and verified, you're now ready to take the next step in securing your blog by enabling HTTPS, ensuring your visitors' data remains private. 

Whether you're sharing your thoughts, showcasing projects, or experimenting with new ideas, your cost-free blog is now set to leave a lasting digital mark. I hope you enjoy this series, and if you have any questions, don't hesitate to reach out.
