---
layout: post
title:  "Enhancing the User Experience for Custom Deployments on Azure Portal"
date:   2023-10-23 10:45:00 +0300
categories: Azure Portal
tags: azure iac bicep azure-portal arm github
---


Deploying infrastructure, Azure resources efficiently is crucial. Azure (and GitHub) offers a powerful solution for this through its Azure Resource Manager (ARM) templates and the "Deploy to Azure" button. In this blog post, we'll explore how you can create a seamless and user-friendly experience for custom deployments on the Azure portal. Let's dive in.

## Custom Azure Deployment "Deploy To Azure" Button

The [Deploy to Azure](https://learn.microsoft.com/azure/azure-resource-manager/templates/deploy-to-azure-button) button is a valuable tool when you want to host your Azure Infrastructure as Code (IaC) artifacts on a public GitHub repository. It simplifies the process of deploying resources to your Azure subscription, allowing for basic parameterization. To add this functionality to your repository's README.md file, you'll need the following:


### An Azure Resource Manager (ARM) Template
The ARM template is a json file, like the `azuredeploy.json` file you can find in the [web-app-sql-database](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.web/web-app-sql-database) of the Azure/azure-quickstart-templates. Just be sure to select the *Raw URL* of the [template](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/web-app-sql-database/azuredeploy.json). 

### A URL Encoded URL of the ARM Template
This URL needs to be URL-encoded to be used. You can use an online encoder or run a PowerShell command as shown below. 

```powershell
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/web-app-sql-database/azuredeploy.json"
[uri]::EscapeDataString($url)

# this will be returned:
# https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fweb-app-sql-database%2Fazuredeploy.json
```
 To have a valid link that will direct you to Azure and start the Custom Deployment process you need to append the URL encoded link to the following base URL **https://portal.azure.com/#create/Microsoft.Template/uri/**. So the valid link will be:

 ```html
 https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fweb-app-sql-database%2Fazuredeploy.json
 ```

### An image used for the Button 
To add the button to your repository, you can use the following image markdown snipet
 
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

The problem with the aforementioned solution, is that as long as the azure template becomes bigger, the UX is a list of numerous parameters stacked one after the other, the makes the user experience not so great. What we want to achieve, is to try to group those parameters together, and give a "wizard-like" deployment experience with distinct steps to the end user. 

Another issue is that the Azure Custom Deployment works only for json ARM templates, while we certainly love to use Bicep (:muscle:) instead for our code. 

In the next sections I will describe the whole workflow that will allow you achieve a better user experience for your custom deployments on Azure. 

### From Bicep to ARM template
We will use as an example a really basic [bicep deployment file](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy.bicep) that creates the following resources:
- App Service Plan and a Web app for containers
- Virtual Network
- Log Analytics Workspace

![Sample Deployment Resources](/images/azure-custom-deployment/sample-deployment-start.jpg)
_Sample Deployment Resources_

The first step is to create a json ARM template out of the Bicep file, which can be done by running `az bicep build`, as shown below.

```bash
 az bicep build --file deploy.bicep --outfile deploy.json
```

Based on what we described in the previous sections, to deploy that template we need to construct the appropriate [URL](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy.json). If you click on that you will be directed to an Azure Portal deployment with a lot of parameters, without any logical grouping, as shown on the image above. Let's see how we can enhance the user experience :rocket:

### Create an Azure Portal form for the deployment template

The goal now is to [create a form](https://learn.microsoft.com/azure/azure-resource-manager/templates/template-specs-create-portal-forms) that appears in the Azure portal to assist users in deploying the Azure template. To achieve that we need a json file, that will describe the form and will define the elements that need to pass from the Portal Form to the Custom deployment file (as prameters). The basic sections of that json file are the 
  - steps, which define the different sections (steps) of the form
  - the Graphical Elements inside that steps, that will help the user to configure the right parameters
  - the outputs where you can define the parameters that need to be passed to the deployment template. 

The basic sections of the file are illutrated below.

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

For really big templates, with countless parameters, it would be painful to manually construct this json representation of the UX form. In order to accelerate this process, you can use the [aka.ms/form/sandbox](https://aka.ms/form/sandbox). Select your json template in the file slector labeled "Deployment Template (optional)", make sure that `Package Type = CustomTemplate` and select "yes" when asked if you want to Auto Generate the Create UI Definition, as shown below. 

![Auto Generate the Create UI Definition](/images/azure-custom-deployment/autoGenerateUIDefiniton.jpg)
_Auto Generate the Create UI Definition_

The [generated UI Definiton json file](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.start.json) will be used as a base for configuring the UI of the form. 

#### Configure the UI Portal form

In the generated json, you can change the `title` of the form; the element is found at  `view > properties > title`. 
You can also shuffle the order of appearenc of the UI elements of the form. The [UI elements](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/create-uidefinition-elements) can be found at  `view > properties > steps > elements`. As you see in the respective [documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/create-uidefinition-elements) there are plenty of UI elements such as, text boxes, drop downs, check boxes, text bloxks etc. 
You can preview the changes you are making, in the [Create UI Definition Sandbox](https://aka.ms/form/sandbox), by pclicking th preview button. Moreover, you can copy elements json snipets, and paste them on your json UI Definiton, to experiment with several UI elements. 

#### Create "Steps" for the UI Portal form
Let's start grouping the bunch of parameters of the [test deployment](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.start.json) given above in two distinct steps. The first step will contain basic settings of the deployed resources, where as the second step will present parameters related to the underlying networking. The way you can organize and present the parameters in the custom UI definiton form, is subjective and is good enough if it serves your goal. 

In order to do that you need to create a second `steps` element, and start moving there, the elements you need to group under networking, such as:
- vnetAddressSpace
- snetDefaultAddressSpace
- snetWebAppAddressSpace
- subnetNameDefault
- subnetNameAsp

![Add Extra Steps](/images/azure-custom-deployment/add-steps.jpg)
_Add Extra Steps_

But this is not really enough. If you try to test now the form it will fail, because the `outputs > parameters` for the UI elements you moved, do not match. You will get an error stating something similar to the below message:

![UI Element cannot be found error](/images/azure-custom-deployment/error-ui-element.jpg)
_UI Element cannot be found error_

To reference the value of an element in the parameters section, you need the whole path, so you need (for all the moved elements) to change the value from `"vnetAddressSpace": "[steps('basics').vnetAddressSpace]"` to `"vnetAddressSpace": "[steps('networking').vnetAddressSpace]"`.

The [new form](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.steps.json) now looks like 

![Sample Deployment step 1](/images/azure-custom-deployment/sample-deployment-step1.jpg)
_Sample Deployment step 1_

![Sample Deployment step 2](/images/azure-custom-deployment/sample-deployment-step2.jpg)
_Sample Deployment step 2_

#### Use functions for further customization
You can further customize the UI form, by using [CreateUiDefinition functions](https://learn.microsoft.com/azure/azure-resource-manager/managed-applications/create-uidefinition-functions). 

> These functions look similar to the ones found in the ARM templates, but they are not the same.
{: .prompt-warning }

For example in our example deployment we have a boolean parameter which sets if we need to deploy Log Analytics Workspace. If we select not to deploy one, then it would be great to hide the parameter `LogAnalyticsWsName`, since there is no need to set a name for a resource not being deployed. To do that you can check the visibility property of the second parameter to be set based on the selected value of the first parameter. This can be done, as shown below, with the usage of the function [equals](https://learn.microsoft.com/azure/azure-resource-manager/managed-applications/create-ui-definition-logical-functions#equals). 

This is the new [file](https://github.com/thotheod/bicep-modules-helper/blob/main/_testModules/deploy-ux.conditional.json), with the conditional visibility of the UI element `LogAnalyticsWsName` parameter.

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
You can also use REST API call to get specific data, such as active subscriprions in your tenant that you have access to, or available regions for deployment of resources, as shown below.

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
To Uue this custom UX form with the “Deploy To Azure” Button you need to construct a link as the following tith four parts
- the URL part for direction to CreateUIDefinition: `https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/`
- `URL_TEMPLATE`: The URL Encoded version of the URI to the remote Azure ARM deployment template
- "redirection" to the uiFormDefinitonUri: `/uiFormDefinitionUri/`
- `CUSTOM_UI_DEF_JSON`: The URL Encoded version of the URI to the custom UI Definition json file

The [actual link for our sample deployment](https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy.json/uiFormDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy-ux.conditional.json) demonstrated in this post, can be constructed as shown below: 

```html

https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/URL_TEMPLATE/uiFormDefinitionUri/CUSTOM_UI_DEF_JSON
<!-- URL_TEMPLATE: The URL Encoded version of the URI to the remote Azure ARM deployment template -->
<!-- CUSTOM_UI_DEF_JSON: The URL Encoded version of the URI to the custom UI Definition json file -->

<!-- The actual link for our sample deployment  -->
https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy.json/uiFormDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fthotheod%2Fbicep-modules-helper%2Fmain%2F_testModules%2Fdeploy-ux.conditional.json

```
