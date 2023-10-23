---
layout: post
title:  "Enhancing the User Experience for Custom Deployments on Azure Portal"
date:   2023-10-23 10:45:00 +0300
categories: Azure Portal
tags: azure iac bicep azure-portal arm github
---


Deploying infrastructure and Azure resources efficiently is crucial. Azure offers a powerful solution for this through its Azure Resource Manager (ARM) templates and the "Deploy to Azure" button. In this blog post, we'll explore how you can create a seamless and user-friendly experience for custom deployments on the Azure portal. Let's dive in.

## Custom Azure Deployment "Deploy To Azure" Button

The [Deploy to Azure](https://learn.microsoft.com/azure/azure-resource-manager/templates/deploy-to-azure-button) button is a valuable tool when you want to host your Azure Infrastructure as Code (IaC) artifacts on a public GitHub repository. It simplifies the process of deploying resources to your Azure subscription, allowing for basic parameterization. To add this functionality to your repository's README.md file, you'll need the following:


### An Azure Resource Manager (ARM) Template
The ARM template is a json file, like the `azuredeploy.json` file you can find in the [web-app-sql-database](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.web/web-app-sql-database) of the Azure/azure-quickstart-templates. Be sure to select the *Raw URL* of the [template](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/web-app-sql-database/azuredeploy.json). 

### A URL Encoded URL of the ARM Template
This URL needs to be URL-encoded to be used. URL encoding ensures that special characters in the URL do not cause issues. You can use an online encoder or run a PowerShell command as shown below. 

```powershell
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/web-app-sql-database/azuredeploy.json"
[uri]::EscapeDataString($url)

# this will be returned:
# https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fweb-app-sql-database%2Fazuredeploy.json
```
 To have a valid link that will direct you to Azure and start the Custom Deployment process, you need to append the URL-encoded link to the following base URL: **https://portal.azure.com/#create/Microsoft.Template/uri/**. So the valid link will be:

 ```html
 https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fweb-app-sql-database%2Fazuredeploy.json
 ```

### An image used for the Button 
To add the button to your repository, you can use the following image markdown snippet
 
 ```markdown
 <!-- Markdown for the Button Image -->
 ![Deploy to Azure](https://aka.ms/deploytoazurebutton)
 ```

The image appears as:

![Deploy to Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)
_Deploy to Azure button_


### A link to the Azure template attached on the button image
To put everything together, you need to add the following markdown snippet 

 ``` markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)]( https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fweb-app-sql-database%2Fazuredeploy.json)
 ```

 One convenient  automation is that all the camel-cased parameter names (e.g. `storageAccountType`) defined in the template will be turned into a space-separated string (`Storage Account Type`) when displayed on the portal.

## Enhanced UX experience

One drawback of the aforementioned solution is that as the Azure template grows in complexity, the user experience (UX) deteriorates. The UX becomes a lengthy list of numerous parameters stacked one after the other, which can be overwhelming for users. What we aim to achieve is a more user-friendly and intuitive experience by grouping these parameters and providing a "wizard-like" deployment process with distinct steps for the end user.

Another challenge is that Azure Custom Deployment currently works only with JSON ARM templates, while many of us prefer using Bicep (:muscle:) for our code.

In the following sections, I will describe the entire workflow that will enable you to create an improved user experience for your custom deployments on the Azure portal.

### From Bicep to ARM template
To illustrate this process, let's consider a basic [bicep deployment file](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy.bicep) that creates the following resources:
- App Service Plan and a Web app for containers
- Virtual Network
- Log Analytics Workspace

![Sample Deployment Resources](/images/azure-custom-deployment/sample-deployment-start.jpg)
_Sample Deployment Resources_

The first step is to convert the Bicep file into a JSON ARM template, which can be done by running the following command:

```bash
 az bicep build --file deploy.bicep --outfile deploy.json
```

As we've described in the previous sections, to deploy this template, you need to construct the appropriate [URL](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy.json). When you click on this URL, it will direct you to an Azure Portal deployment with numerous parameters, without any logical grouping, as shown in the image above. Now, let's explore how to enhance the user experience. :rocket:

### Create an Azure Portal form for the deployment template

The goal now is to [create a form](https://learn.microsoft.com/azure/azure-resource-manager/templates/template-specs-create-portal-forms) that appears in the Azure portal to assist users in deploying the Azure template. This form is intended to provide a more user-friendly interface for deploying Azure resources compared to the standard parameter input.

To achieve this, you'll need a JSON file that describes the form and defines the elements that need to pass from the Portal Form to the Custom deployment file as parameters. The basic sections of this JSON file are:
  - steps, which define the different sections (steps) of the form
  - the Graphical Elements inside that steps, that will help the user to configure the right parameters
  - the outputs where you can define the parameters that need to be passed to the deployment template. 

and they are illustrated below: 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2021-09-09/uiFormDefinition.schema.json",
  "view": {
    "kind": "Form",
    "properties": {
      "title": "Test Form View",
      "steps": [
        {
          "name": "basics",
          "label": "Basics",
          "elements": [
            ...
          ]
        }
      ]
    },
    "outputs": {
      ...
    }
  }
}
```

For very large templates with numerous parameters, manually constructing this JSON representation of the UX form can be daunting. To expedite this process, you can use the [aka.ms/form/sandbox](https://aka.ms/form/sandbox). Select your json template in the file selector labeled "Deployment Template (optional)", make sure that `Package Type = CustomTemplate` and select "yes" when asked if you want to Auto Generate the Create UI Definition, as shown below. 

![Auto Generate the Create UI Definition](/images/azure-custom-deployment/autoGenerateUIDefiniton.jpg)
_Auto Generate the Create UI Definition_

The [generated UI Definiton json file](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.start.json) will be used as a base for configuring the UI of the form. 

#### Configuring the UI Portal form

In the generated JSON, you can change the form's `title`, which can be found at `view > properties > title`. 
You can also rearrange the order in which the UI elements appear in the form. The [UI elements](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/create-uidefinition-elements) can be found at  `view > properties > steps > elements`. As you see in the [documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/create-uidefinition-elements) there are various UI elements available, including text boxes, drop-downs, checkboxes, and text blocks. You can preview the changes you make in the [Create UI Definition Sandbox](https://aka.ms/form/sandbox), by clicking the preview button. Additionally, you can copy element JSON snippets and paste them into your JSON UI Definition to experiment with different UI elements. 

#### Creating "Steps" for the UI Portal form
Let's start by grouping the parameters from the [test deployment](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.start.json) given above in two distinct steps. The first step will encompass the basic settings for the deployed resources, while the second step will focus on parameters related to the underlying networking. The way you organize and present these parameters in the custom UI definition form is subjective and is adequate as long as it aligns with your goals.

To achieve this, create a second `steps` element and start moving the elements you wish to group under networking, such as:

- vnetAddressSpace
- snetDefaultAddressSpace
- snetWebAppAddressSpace
- subnetNameDefault
- subnetNameAsp

![Add Extra Steps](/images/azure-custom-deployment/add-steps.jpg)
_Add Extra Steps_

However, this alone is not sufficient. If you try to test the form now, it will fail because the `outputs > parameters` for the UI elements you moved do not match. You will encounter an error message similar to the one below:

![UI Element cannot be found error](/images/azure-custom-deployment/error-ui-element.jpg)
_UI Element cannot be found error_

To reference the value of an element in the parameters section, you need to provide the entire path. For all the moved elements, change the value from `"vnetAddressSpace": "[steps('basics').vnetAddressSpace]"` to `"vnetAddressSpace": "[steps('networking').vnetAddressSpace]"`.

The [new form](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.steps.json) now looks like this:

![Sample Deployment step 1](/images/azure-custom-deployment/sample-deployment-step1.jpg)
_Sample Deployment step 1_

![Sample Deployment step 2](/images/azure-custom-deployment/sample-deployment-step2.jpg)
_Sample Deployment step 2_

#### Using functions for further customization
ou can enhance the customization of the UI form by utilizing [CreateUiDefinition functions](https://learn.microsoft.com/azure/azure-resource-manager/managed-applications/create-uidefinition-functions). 

> These functions look similar to the ones found in the ARM templates, but they are not the same.
{: .prompt-warning }

For example, in our sample deployment, we have a boolean parameter that determines whether to deploy a Log Analytics Workspace. If we choose not to deploy one, it would be logical to hide the parameter `LogAnalyticsWsName` since there is no need to set a name for a resource that won't be deployed. To achieve this, you can adjust the visibility property of the second parameter based on the selected value of the first parameter. This can be done using the function [equals](https://learn.microsoft.com/azure/azure-resource-manager/managed-applications/create-ui-definition-logical-functions#equals). 

The resulting [file](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.conditional.json), includes the conditional visibility of the UI element `LogAnalyticsWsName` parameter.

``` json

{
    ...
    "steps": [
        ...
                {
                    "name": "basics",
                    "label": "Basics",
                    "elements": [
                        ...
                         {
                            "name": "logAnalyticsWsName",
                            "type": "Microsoft.Common.TextBox",
                            "label": "Log Analytics Workspace Name",
                            "subLabel": "",
                            "defaultValue": "logs-webapp-sample",
                            "toolTip": "",
                            "constraints": {
                                "required": false,
                                "regex": "",
                                "validationMessage": "",
                                "validations": []
                            },
                            "infoMessages": [],
                            // change the visible property from true|false
                            // "visible": true
                            // to the following, by using the equals function
                            "visible": "[equals(steps('basics').deployLogAnalyticsWs, true)]"
                        },    
                        ...
                    ] 
                }
            ]       
}

```

#### Advanced Customization
For more advanced customization, you can employ REST API calls to retrieve specific data, such as active subscriptions in your tenant that you have access to or available regions for resource deployment, as demonstrated below:

``` json

{
    ...
    "steps": [
        ...
                {
                    "name": "basics",
                    "label": "Basics",
                    "elements": [
                        ...
                         {
                            "name": "getSubscriptions",
                            "type": "Microsoft.Solutions.ArmApiControl",
                            "request": {
                                "method": "POST",
                                "path": "providers/Microsoft.ResourceGraph/resources?api-version=2021-03-01",
                                "body": {
                                    "query": "ResourceContainers | where type =~ 'microsoft.resources/subscriptions' | where properties.state =~ 'enabled' | project label=tostring(name), description=subscriptionId, value=subscriptionId | order by label asc"
                                }
                            }
                        },
                        {
                            "name": "getLocations",
                            "type": "Microsoft.Solutions.ArmApiControl",
                            "request": {
                                "method": "GET",
                                "path": "locations?api-version=2019-11-01"
                            }
                        },   
                        ...
                    ] 
                }
            ]       
}

```


## “Deploy To Azure” Button with Custom UI Definition Form
To use this custom UX form with the "Deploy To Azure" Button, you need to construct a link with four parts:
- The URL part for directing to the CreateUIDefinition: `https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/`
- `URL_TEMPLATE`: The URL-encoded version of the URI to the remote Azure ARM deployment template
- A "redirection" to the uiFormDefinitonUri: `/uiFormDefinitionUri/`
- `CUSTOM_UI_DEF_JSON`: The URL-encoded version of the URI to the custom UI Definition JSON file.

The [actual link for our sample deployment](https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy.json/uiFormDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy-ux.conditional.json) demonstrated in this post, can be constructed as shown below: 

```html

https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/URL_TEMPLATE/uiFormDefinitionUri/CUSTOM_UI_DEF_JSON
<!-- URL_TEMPLATE: The URL Encoded version of the URI to the remote Azure ARM deployment template -->
<!-- CUSTOM_UI_DEF_JSON: The URL Encoded version of the URI to the custom UI Definition json file -->

<!-- The actual link for our sample deployment  -->
https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy.json/uiFormDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy-ux.conditional.json

```
This link allows you to integrate the custom UI definition form seamlessly with the "Deploy To Azure" Button, enhancing the deployment experience for your users. If you're looking for more advanced examples of this approach, you can explore the following resources:
- [ACA LZA](https://github.com/Azure/aca-landing-zone-accelerator/tree/main/scenarios/aca-internal/azure-resource-manager) 
- [App Service LZA](https://github.com/Azure/appservice-landing-zone-accelerator/tree/main/scenarios/secure-baseline-multitenant/azure-resource-manager)

Enhancing user experiences for custom deployments on Azure has never been easier. With these tools and strategies at your disposal, you can streamline the deployment process and provide users with a more intuitive and user-friendly way to configure and launch resources in the Azure cloud. Whether you're working with ARM templates, Bicep, or custom UI definitions, the goal is to make Azure deployment as smooth as possible, and these techniques can help you achieve that.
