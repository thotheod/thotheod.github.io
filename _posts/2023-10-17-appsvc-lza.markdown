---
layout: post
title:  "What is App Service LZA and how it can help me"
date:   2023-10-17 11:45:00 +0300
categories: Azure IaC
tags: azure iac bicep app-service architecture
---
In the [previous post](/posts/aca-lza/) we examined what a [Landing Zone Accelerator](https://aka.ms/lza) is, and specifically we analyzed the [Azure Container Apps LZA](https://aka.ms/lza). Let's continue with a brief introduction to App Service LZA [aka.ms/EnterpriseScale-AppService](https://aka.ms/EnterpriseScale-AppService)
nimbus-musings.com

The Azure App Service landing zone accelerator is an open-source collection of architectural guidance and reference implementation to accelerate deployment of Azure App Service at scale. Latest release [v1.0.2](https://github.com/Azure/appservice-landing-zone-accelerator/releases/tag/v1.0.2) covers two different scenarios.

## Scenario 1: Multi-tenant App Service Secure Baseline
 This reference architecture shows how to run a web application workload on Azure App Service Plan in a secure configuration. This secure baseline follow [Defense in Depth](https://learn.microsoft.com/shows/azure-videos/defense-in-depth-security-in-azure) approach to protect App Service workload against cloud vulnerabilities along with additional Well-Architected Framework pillars to enable a resilient solution. 

 The main characteristics of this scenario are:
- Use web apps with VNet Integration and private endpoint. No public IP to the web apps.
- Use app service deployment slots for enabling Blue-Green Deployment scenarios
- Secure all subnets with [Network Security Groups(NSG)](https://learn.microsoft.com/azure/container-apps/firewall-integration#nsg-allow-rules)
- Control egress traffic with [Azure Firewall](https://learn.microsoft.com//azure/container-apps/user-defined-routes). The benefits of such a setup are numerous; to name some:
  - **Data protection / data exfiltration prevention**: You can prevent malware or malicious software within your environment from communicating with external command and control servers. This can help in identifying and stopping cyber threats before they cause significant damage)
  - **Compliance**: regulatory frameworks, such as GDPR, HIPAA, and PCI DSS, require organizations to have controls in place to monitor and restrict outbound traffic
  - **Access Control**: Egress control allows you to specify which applications and services can access external resources, reducing the attack surface and limiting the potential for exposure to vulnerabilities in those applications)
  - **Visibility and Monitoring**: Azure Firewall provides the ability to log and analyze outbound traffic. This visibility is essential for detecting suspicious activities, security incidents, and compliance auditing.
  - **Centralized Management**: Azure Firewall allows for centralized management of egress traffic policies, making it easier to maintain and enforce consistent security controls across the entire environment.
- Expose web applications / APIs through an Application Reverse Proxy with WAF (Layer 7) capabilities, such as  [Azure Front Door Premium](https://learn.microsoft.com/azure/frontdoor/front-door-cdn-comparison)
- Use Azure native PaaS services for supporting resources required by your application (i.e. Databases) which are exposed only through Private Endpoints
- Use [Azure Key Vault](https://learn.microsoft.com/azure/key-vault/general/basic-concepts) to securely store and access secrets, such as API keys, passwords, certificates, connection strings and other.
- Use [User Assigned Managed Identities](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types) for securing with fine grain access of the web apps to other resources (i.e. Azure Key Vault, databases etc)
- Use [Azure Bastion Services](https://learn.microsoft.com/azure/bastion/bastion-overview) to securely connect to private Jump-boxes or self-hosted devops agents, to deploy your applications. 

![Multitenant App Service LZA](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Images/Multitenant/AppServiceLandingZoneArchitecture-multitenant.png?raw=true){: width="700" height="400" }
_Multitenant App Service LZA_

## Scenario 2: Line of Business application using internal App Service Environment v3
This scenario covers how to run a web-app workload for line of business applications on Azure App Services Environment (ASEv3), in an internal configuration. The functionality is similar to the first scenario, with the difference that all the web apps are not exposed by any resource to the public internet.

> This scenario is soon to be set obsolete. The Usage of ASEv3 (or multitenant app service plan) will be integrated in the first scenario, in the form of a feature flag.
{: .prompt-warning  }

![ASEv3 scenario - App Service LZA](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Images/ASE/AppServiceLandingZoneArchitecture.png?raw=true){: width="700" height="400" }
_ASEv3 scenario - App Service LZA_


## Design areas
The architecture is considered across six key design areas. Please review them as part of your overall understanding of this landing zone accelerator.

- [Identity and Access Management](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Design-Areas/identity-access-mgmt.md)
- [Network Topology and Connectivity](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Design-Areas/networking.md)
- [Management and Monitoring](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Design-Areas/mgmt-monitoring.md)
- [Business Continuity and Disaster Recovery](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Design-Areas/BCDR.md)
- [Security, Governance, and Compliance](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Design-Areas/security-governance-compliance.md)
- [Application Automation and DevOps](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Design-Areas/automation-devops.md)

## Infrastructure as Code (IaC) options
The Reference Implementation is supporting two options:
- [Bicep](https://github.com/Azure/appservice-landing-zone-accelerator/tree/main/scenarios/secure-baseline-multitenant/bicep)
- [Terraform](https://github.com/Azure/appservice-landing-zone-accelerator/tree/main/scenarios/secure-baseline-multitenant/terraform)

Both implementations follow best practices, as described in the [contributing](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/CONTRIBUTING.md) document. Special care has been given to:
- Cloud Adoption Framework [Naming Convention](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) which is using in an easy way [Azure Resource Abbreviations](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations) and [Naming rules and restrictions](https://learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules) for Azure resources, so that we avoid deployment failures and inconsistency in the naming of the resources
- Modularized approach for improved readability of the code and for easier customization
- Usage of well established building blocks, such as the ones presented in [Common Azure Resource Modules Library](https://github.com/Azure/ResourceModules)

### 'Deploy To Azure' Button
The quickest method to deploy the LZA is to click on the ["Deploy To Azure"](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/scenarios/secure-baseline-multitenant/README.md#quick-deployment-to-azure) button.

This will redirect you to the Azure Portal, where a series of steps will guide you through the customization of the deployment based on your needs. These steps include:

- **Deployment Settings (Basics)**: In this deployment step, fill in basic parameters such as Deployment Region and Workload Name
- **Network Settings**: Adjust the default IPv4 address spaces for all required subnets if necessary.
- **Jump-Box Settings**: Provide details if you need a Jumpbox VM, including VM SKU, username, and password.
- **Azure SQL Settings**: In this step you select if you want an SQL Server to be deployed in the spoke Resource Group. If you select yes, then you are given the option to select the SQL Server Authentication method (Azure AD, or SQL Server local User)
- **Deployment feature flags**: the final deployment by adding extra resources like Redis Cache, Bastion Service, etc., using simple true/false parameters.
