---
title: Use API version profiles with .NET in Azure Stack | Microsoft Docs
description: Learn how to use API version profiles with .NET SDK in Azure Stack.
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''

ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/17/2019
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 05/16/2019

---

# Use API version profiles with .NET in Azure Stack

*Applies to: Azure Stack integrated systems and Azure Stack Development Kit*

The .NET SDK for the Azure Stack Resource Manager provides tools to help you build and manage your infrastructure. Resource providers in the SDK include Compute, Networking, Storage, App Services, and [Key Vault](/azure/key-vault/key-vault-whatis). The .NET SDK includes 14 NuGet packages. You must download these packages to your solution every time you compile your project. However, you can specifically download which resource provider you'll use for the **2019-03-01-hybrid** or **2018-03-01-hybrid** versions in order to optimize the memory for your app. Each package consists of a resource provider, the respective API version, and the API profile to which it belongs. API profiles in the .NET SDK enable hybrid cloud development by helping you switch between global Azure resources and resources on Azure Stack.

## .NET and API version profiles

An API profile is a combination of resource providers and API versions. Use an API profile to get the latest, most stable version of each resource type in a resource provider package.

- To make use of the latest versions of all the services, use the **latest** profile of the packages. This profile is part of the **Microsoft.Azure.Management** NuGet package.

- To use the services compatible with Azure Stack, use one of the following packages:
  - **Microsoft.Azure.Management.Profiles.hybrid\_2019\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg** 
  - **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**

  Ensure that the **ResourceProvider** portion of the above NuGet package is changed to the correct provider.

- To use the latest API version of a service, use the **Latest** profile of the specific NuGet package. For example, if you want to use the **latest-API** version of the Compute service alone, use the **latest** profile of the **Compute** package. The **latest** profile is part of the **Microsoft.Azure.Management** NuGet package.

- To use specific API versions for a resource type in a specific resource provider, use the specific API versions defined inside the package.

You can combine all of the options in the same application.

## Install the Azure .NET SDK

- Install Git. For instructions, see [Getting Started - Installing Git][].

- To install the correct NuGet packages, see [Finding and installing a package][].

- The packages that need to be installed depend on the profile version you want to use. The package names for the profile versions are:

   - **Microsoft.Azure.Management.Profiles.hybrid\_2019\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**

   - **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**

- To install the correct NuGet packages for Visual Studio Code, see the following link to download the [NuGet Package Manager instructions][].

- If not available, create a subscription and save the subscription ID to be used later. For information about how to create a subscription, see [Create subscriptions to offers in Azure Stack][].

- Create a service principal and save the client ID and the client secret. For information about how to create a service principal for Azure Stack, see [Provide applications access to Azure Stack][]. The client ID is also known as the application ID when creating a service principal.

- Make sure your service principal has the contributor/owner role on your subscription. For information about how to assign a role to service principal, see [Provide applications access to Azure Stack][].

## Prerequisites

To use the .NET Azure SDK with Azure Stack, you must supply the following values, and then set the values with environment variables. To set the environmental variables, see the instructions following the table for your specific operating system.

| Value                     | Environment variables   | Description                                                                                                             |
|---------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------------|
| Tenant ID                 | `AZURE_TENANT_ID `      | The value of your Azure Stack [*tenant ID*][].                                                                          |
| Client ID                 | `AZURE_CLIENT_ID `      | The service principal app ID saved when the service principal was created in the previous section of this article. |
| Subscription ID           | `AZURE_SUBSCRIPTION_ID` | The [*subscription ID*][] is how you access offers in Azure Stack.                                                      |
| Client Secret             | `AZURE_CLIENT_SECRET`   | The service principal app secret saved when the service principal was created.                                      |
| Resource Manager Endpoint | `ARM_ENDPOINT`          | See [*the Azure Stack Resource Manager endpoint*][].                                                                    |
| Location                  | `RESOURCE_LOCATION`     | Location of Azure Stack.

To find the tenant ID for your Azure Stack, follow the instructions [in this article](../operator/azure-stack-csp-ref-operations.md). To set your environment variables, do the following:

### Windows

To set the environment variables in a Windows command prompt, use the following format:

```shell
Set Azure_Tenant_ID=Your_Tenant_ID
```

### MacOS, Linux, and Unix-based systems

In Unix-based systems, use the following command:

```shell
Export Azure_Tenant_ID=Your_Tenant_ID
```

### The Azure Stack Resource Manager endpoint

Azure Resource Manager is a management framework that enables administrators to deploy, manage, and monitor Azure resources. Azure Resource Manager can handle these tasks as a group, rather than individually, in a single operation.

You can get the metadata info from the Resource Manager endpoint. The endpoint returns a JSON file with the info required to run your code.

Note the following considerations:

- The **ResourceManagerUrl** in the Azure Stack Development Kit (ASDK) is: https://management.local.azurestack.external/.

- The **ResourceManagerUrl** in integrated systems is: `https://management.<location>.ext-<machine-name>.masd.stbtest.microsoft.com/`.
To retrieve the metadata required: `<ResourceManagerUrl>/metadata/endpoints?api-version=1.0`.

Sample JSON file:

```json
{
   "galleryEndpoint": "https://portal.local.azurestack.external:30015/",
   "graphEndpoint": "https://graph.windows.net/",
   "portal Endpoint": "https://portal.local.azurestack.external/",
   "authentication": 
      {
         "loginEndpoint": "https://login.windows.net/",
         "audiences": ["https://management.yourtenant.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
      }
}
```

## Existing API profiles

- **Microsoft.Azure.Management.Profiles.hybrid\_2019\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**: Latest profile built for Azure Stack. Use this profile for services to be most compatible with Azure Stack, as long as you're on version 1904 or later.

- **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.<*ResourceProvider*>.0.9.0-preview.nupkg**: Use this profile for services to be compatible with Azure Stack for versions 1808 or later.

- **Latest**: Profile consisting of the latest versions of all services. Use the latest versions of all the services. This profile is part of the **Microsoft.Azure.Management** NuGet package.

For more information about Azure Stack and API profiles, see the [Summary of API profiles][].

## Azure .NET SDK API profile usage

Use the following code to instantiate a resource management client. Similar code can be used to instantiate other resource provider clients (such as Compute, Network, and Storage).

```csharp
var client = new ResourceManagementClient(armEndpoint, credentials)
{
    SubscriptionId = subscriptionId;
};
```

The `credentials` parameter in this code is required to instantiate a client. The following code generates an authentication token by the tenant ID and the service principal:

```csharp
var azureStackSettings = getActiveDirectoryServiceSettings(armEndpoint);
var credentials = ApplicationTokenProvider.LoginSilentAsync(tenantId, servicePrincipalId, servicePrincipalSecret, azureStackSettings).GetAwaiter().GetResult();
```

The `getActiveDirectoryServiceSettings` call in the code retrieves Azure Stack endpoints from the metadata endpoint. It states the environment variables from the call that's made:

```csharp
public static ActiveDirectoryServiceSettings getActiveDirectoryServiceSettings(string armEndpoint)
{
    var settings = new ActiveDirectoryServiceSettings();
    try
    {
        var request = (HttpWebRequest)HttpWebRequest.Create(string.Format("{0}/metadata/endpoints?api-version=1.0", armEndpoint));
        request.Method = "GET";
        request.UserAgent = ComponentName;
        request.Accept = "application/xml";
        using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
        {
            using (StreamReader sr = new StreamReader(response.GetResponseStream()))
            {
                var rawResponse = sr.ReadToEnd();
                var deserialized = JObject.Parse(rawResponse);
                var authenticationObj = deserialized.GetValue("authentication").Value<JObject>();
                var loginEndpoint = authenticationObj.GetValue("loginEndpoint").Value<string>();
                var audiencesObj = authenticationObj.GetValue("audiences").Value<JArray>();
                settings.AuthenticationEndpoint = new Uri(loginEndpoint);
                settings.TokenAudience = new Uri(audiencesObj[0].Value<string>());
                settings.ValidateAuthority = loginEndpoint.TrimEnd('/').EndsWith("/adfs", StringComparison.OrdinalIgnoreCase) ? false : true;
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(String.Format("Could not get AD service settings. Exception: {0}", ex.Message));
    }
    return settings;
}
```

These steps enable you to use the API profile NuGet packages to deploy your application successfully to Azure Stack.

## Samples using API Profiles

You can use the following samples as a reference for creating solutions with .NET and the Azure Stack API profiles:

- [Manage Resource Groups](https://github.com/Azure-Samples/hybrid-resources-dotnet-manage-resource-group)
- [Manage Storage Accounts](https://github.com/Azure-Samples/hybird-storage-dotnet-manage-storage-accounts)
- [Manage a Virtual Machine](https://github.com/Azure-Samples/hybrid-compute-dotnet-manage-vm): This sample uses the **2019-03-01-hybrid** profile supported by Azure Stack.

## Next steps

Learn more about API profiles:

- [Manage API version profiles in Azure Stack](azure-stack-version-profiles.md)
- [Resource provider API versions supported by profiles](azure-stack-profiles-azure-resource-manager-versions.md)

  [Getting Started - Installing Git]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
  [Finding and installing a package]: /nuget/tools/package-manager-ui
  [NuGet Package Manager instructions]: https://marketplace.visualstudio.com/items?itemName=jmrog.vscode-nuget-package-manager
  [Create subscriptions to offers in Azure Stack]: ../operator/azure-stack-subscribe-plan-provision-vm.md
  [Provide applications access to Azure Stack]: ../operator/azure-stack-create-service-principals.md
  [*tenant ID*]: ../operator/azure-stack-identity-overview.md
  [*subscription ID*]: ../operator/azure-stack-plan-offer-quota-overview.md#subscriptions
  [*the Azure Stack Resource Manager endpoint*]: ../user/azure-stack-version-profiles-ruby.md#the-azure-stack-resource-manager-endpoint
  [Summary of API profiles]: ../user/azure-stack-version-profiles.md#summary-of-api-profiles
  [Test Project to Virtual Machine, vNet, resource groups, and storage account]: https://github.com/seyadava/azure-sdk-for-net-samples/tree/master/TestProject
  [Use Azure PowerShell to create a service principal with a certificate]: ../operator/azure-stack-create-service-principals.md
  [Run unit tests with Test Explorer.]: /visualstudio/test/run-unit-tests-with-test-explorer?view=vs-2017
