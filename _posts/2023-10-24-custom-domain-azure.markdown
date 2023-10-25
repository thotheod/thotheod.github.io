---
layout: post
title:  "Do you have Azure Subscription? Go grab a custom domain!"
date:   2023-10-24 11:45:00 +0300
categories: Azure app-innovation
tags: azure app-innovation app-service custom-domain
---

If you're a software developer working with Azure technologies or are employed by an organization that provides Visual Studio Enterprise to its employees, you might already have access to the valuable [Azure Visual Studio Enterprise Subscription](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/). This subscription offers a substantial monthly credit allocation, ranging from $50 to $150 (at the time of writing), depending on your subscription type. These credits are at your disposal for individual development and testing on the Azure platform. In this blog post, we'll explore how you can make the most of this opportunity and acquire a custom domain for development and testing purposes. This will provide a more accurate representation of the production environment, allowing you to identify and address potential issues early in the development process and can lead to a more reliable and secure application when it's deployed to a live environment.

## App Service Domain

App Service domains are custom domains that can be conveniently managed within the Azure environment. While their primary purpose is to simplify the management of custom domains for Azure App Service, they can also be effectively utilized for various testing and development purposes.

Behind the scenes, Azure seamlessly handles the registration of custom domains through a third-party domain Domain Registrar, such as domainsbyproxy.com. The beauty of it is that all domain management and billing are transparently handled within the Azure platform.

As of the time of writing, the  [annual cost](https://azure.microsoft.com/pricing/details/app-service/windows/) for App Service domains is approximately $12, making them an accessible and practical choice for developers and organizations looking to enhance their testing and development experiences.

### Create a new App Service Domain

1. Open your Azure Portal and on the top "Search Box" write "app service domain", and select from the suggestion list the service `App Service Domains`.
   
![Search for App Service Domain](/images/custom-domain-azure/01-app-svc-domain.jpg){: width="400" height="200" }
_Search for App Service Domain_

2. If you have already one ore more App Service Domains, you can find them in this list. In any case, click on the **+Create** button to create a new one. Select (or create a new one) a Resource Group, and on the Domain Details test you desired domain for availability. If the domain you are testing is not available, you will be given alternative suggestions. 
   
![Fill In Details](/images/custom-domain-azure/02-app-svc-domain-validation.jpg){: width="400" height="500" }
_Fill In Details for the App Service Domain_

3. Go through all the steps, fill in any information needed and create the new Custom Domain. Once everything finishes, you will find two new resources in the Resource Group you selected
   - The Actual App Service Domain
   - A DNS Zone
  
  ![Custom Domain related resources](/images/custom-domain-azure/04-app-svc-domain-resources.jpg){: width="800" height="250" }
_Custom Domain related resources_


> The top level domains that will be available are com, net, co.uk, org, nl, in, biz, org.uk, and co.in.
{: .prompt-info }

### Manage the Custom Domain
If you click on the custom domain, you can go to the "Domain Management" area where you can change the domain renewal process to automatic or manual. You can also update domain related information (such as contact information etc) and you can see when is its expiration dat.

  ![Custom Domain related resources](/images/custom-domain-azure/05-app-svc-domain-revewal.jpg){: width="800" height="300" }
_Custom Domain related resources_

### Manage the DNS Zone
As you have already noticed, together with the Custom Domain, you get a DNS Zone, which is used to host the DNS records for the particular domain. In the DNS Zone, you can add any DNS Record required for your scenario, such as A Records, CNAMEs, or TXT records for Domain verification purposes. Additionally you can create [child DNS zones](https://azure.microsoft.com/updates/azure-dns-introducing-automatic-child-zone-delegation/) which are automatically attached to parent zone.

![Child Domain Zones](/images/custom-domain-azure/06-app-svc-domain-zone-sub.jpg){: width="400" height="250" }
_Child Domain Zones_