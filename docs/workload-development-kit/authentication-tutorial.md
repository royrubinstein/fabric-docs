---
title: Fabric Workload Development Kit authentication setup (preview)
description: Learn how to set up the authorization for a customized Fabric workload.
author: KesemSharabi
ms.author: kesharab
ms.topic: how-to
ms.custom:
ms.date: 07/14/2024
---

# Authentication setup (preview)

> [!NOTE]  
> To configure the following authentication settings in the setup guide, a Global Administrator role is required.

In order to be able to work with authentication, you need to set up its three component parts:
* [Microsoft Entra ID Application](/power-bi/developer/visuals/entra-id-authentication)
* [Front-end sample](./extensibility-front-end.md)
* [Back-end sample](./extensibility-back-end.md)

To work with authentication in Fabric, follow this guide.

## Azure storage service provisioning

The sample demonstrates how to store and read data from and to lakehouses. It requires generating tokens for the Azure Storage service in OBO flows. To generate tokens, users need to consent to the application using Azure Storage. In order to consent, Azure Storage needs to be provisioned in the tenant.

To make sure Azure Storage is provisioned in the tenant:

1. Log into the [Azure portal](https://portal.azure.com)
1. Go to **Microsoft Entra ID** > **Enterprise applications**
1. In the filters, choose **application type = all applications**. The application ID starts with e406a681-f3d4-42a8-90b6-c2b029497af1

    :::image type="content" source="./media/authentication-tutorial/azure-storage-provisioning.png" alt-text="Screenshot showing Azure Storage provisioning." lightbox="./media/authentication-tutorial/azure-storage-provisioning.png":::

If you see the Azure Storage application, this means was already provisioned and you can continue to the [next step](#configure-your-application-in-microsoft-entra-id-manually). If not, a Global Administrator needs to configure it.

Open **Windows PowerShell** as administrator and run the following script:
  
```console
Install-Module az  
Import-Module az  
Connect-AzureAD  
New-AzureADServicePrincipal -AppId e406a681-f3d4-42a8-90b6-c2b029497af1
```

## Configure your application in Microsoft Entra ID manually 

To work with authentication, you need an application registered in Microsoft Entra ID. If you don't have an application registered, follow [this guide](/entra/identity-platform/quickstart-register-app#register-an-application) to create a new application.

1. Apply the following configurations to your application:

   * Make the application a multitenant app.
   * For dev applications, configure the redirect URI as `http://localhost:60006/close` with SPA platform. This configuration is required for our consent support. You can add other redirect URIs as desired.
  
   > [!NOTE]
   >
   >* The redirect URI should be a URI that simply closes the page when navigating to it. The URI `http://localhost:60006/close` is already configured in the frontend sample and you can change it in [Frontend/src/index.ts](https://github.com/microsoft/Microsoft-Fabric-workload-development-sample/blob/main/Frontend/src/index.ts) (If you change it, make sure it matches the one configured for your application).
   >* You can configure the redirect URI after creating the application from the **Manage** menu under **Authentication**.
   >* The Redirect URL must return an HTML page that just calls to JS “windows.close()”.

   :::image type="content" source="./media/authentication-tutorial/register-application.png" alt-text="Screenshot of application registration UI." lightbox="./media/authentication-tutorial/register-application.png":::

1. Change the *Application ID URI* for your application. Go to **Manage** > **Expose an API**, and edit the Application ID URI for your app:

   For the development scenario, the Application ID URI should start with: `api://localdevinstance/<Workload publisher's tenant ID in lower case (the tenant ID of the user used in Fabric to run the sample)>/<Name of your workload>` and an optional subpath at the end that starts with `/` (see examples).

   Where:

   * The workload name is exactly as specified in the manifest.
   * The ID URI doesn't end with a slash.
   * At the end of the *ID URI* there can be an optional subpath consisting of a string of English lower or upper case letters, numbers, and dashes, up to 36 characters.

   > [!TIP]
   > For help finding the tenant ID, see [How to find your Microsoft Entra tenant ID](/entra/fundamentals/how-to-find-tenant).

   For example, if the publisher's tenant ID is *853d9f4f-c71b-4420-b6ec-60e503458946*, and the workload's name is *Fabric.WorkloadSample* then:

   * The following URIs *are* valid

      * `api://localdevinstance/853d9f4f-c71b-4420-b6ec-60e503458946/Fabric.WorkloadSample`
      * `api://localdevinstance/853d9f4f-c71b-4420-b6ec-60e503458946/Fabric.WorkloadSample/abc`

   * The following URIs *aren't* valid

      * `api://localdevinstance/853d9f4f-c71b-4420-b6ec-60e503458946/Fabric.WorkloadSample/af/`
      * `api://localdevinstance/853d9f4f-c71b-4420-b6ec-60e503458946/Fabric.WorkloadSample/af/a`
      * Any ID URI that doesn't start with `api://localdevinstance/853d9f4f-c71b-4420-b6ec-60e503458946/Fabric.WorkloadSample`

### Add a scope for CRUD/jobs

To work with Create, Read, Update and Delete APIs for workload items, and perform other operations with jobs, [add a scope](/entra/identity-platform/quickstart-configure-app-expose-web-apis#add-a-scope). In addition, add two dedicated Fabric applications to the preauthorized applications for that scope to indicate that your API (the scope you created) trusts Fabric:

* Under **Expose an API**, select **Add a scope**. Name the scope *FabricWorkloadControl* and provide the necessary details for it.

* Under **Authorized client applications**, select **Add a client application**. Add `d2450708-699c-41e3-8077-b0c8341509aa` (Fabric client for workloads application) and select your scope.

### Add scopes for data plane API

Other scopes need to be registered to represent groups of operations exposed by the data plane API.
In the backend sample, we provide four examples. You can see them in [Backend/src/Constants/scopes.cs](https://github.com/microsoft/Microsoft-Fabric-workload-development-sample/blob/main/Backend/src/Constants/WorkloadScopes.cs). The scopes are:

* `Item1.Read.All`: Used for reading workload items
* `Item1.ReadWrite.All`: Used for reading/writing workload items
* `FabricLakehouse.Read.All`: Used for reading lakehouse files
* `FabricLakehouse.ReadWrite.All`: Used for reading/writing lakehouse files  

Preauthorize `871c010f-5e61-4fb1-83ac-98610a7e9110` (the Fabric client application) for these scopes.

The application IDs of these apps can be found under [*Microsoft Power BI* and *Power BI Service*](/troubleshoot/azure/entra/entra-id/governance/verify-first-party-apps-sign-in#application-ids-of-commonly-used-microsoft-applications).  

Here's how your *Expose an API* section should look in your application. In this example, the ID URI is `api://localdevinstance/853d9f4f-c71b-4420-b6ec-60e503458946/Fabric.WorkloadSample`:

:::image type="content" source="./media/authentication-tutorial/expose-api-section.png" alt-text="Screenshot showing how your Expose an API section should look." lightbox="./media/authentication-tutorial/expose-api-section.png":::

### Generate a secret for your application

Under **Certificates & secrets**, select the **Secrets** tab and add a secret. Give it any name you want and save it. Use this secret when configuring the [backend sample](extensibility-back-end.md).

:::image type="content" source="./media/authentication-tutorial/generate-secrets-dialog.png" alt-text="Screenshot of generate secrets dialog.":::

### Add optional claim 'idtyp'

Under **Token configuration**, select **Add optional claim**. Choose **Access token** and add idtyp.

:::image type="content" source="./media/authentication-tutorial/add-claim-idtyp.png" alt-text="Screenshot showing adding claim idtyp." lightbox="./media/authentication-tutorial/add-claim-idtyp.png":::

### Add API permissions

Under **API permissions**, add the desired permissions for your application. For the backend sample, add **Storage user_impersonation** (for OneLake APIs) and **Power BI Workspace.Read.all** (for workload control APIs):

:::image type="content" source="./media/authentication-tutorial/add-api-permissions.png" alt-text="Screenshot showing adding API permissions." lightbox="./media/authentication-tutorial/add-api-permissions.png":::

To learn more about API permissions, see [Update an app's requested permissions in Microsoft Entra ID](/entra/identity-platform/howto-update-permissions).

### Make sure your application is set to work with auth token v1

Under **Manifest**, make sure `accessTokenAcceptedVersion` is set to either null or "1".

## Configure your application in Microsoft Entra Identity automatically by using a script

For a streamlined setup of your application in Microsoft Entra Identity, you can opt to use an automated PowerShell script. To configure your application, follow these steps:

1. **Install Azure CLI**: Begin by installing the Azure Command-Line Interface (CLI) [Install the Azure CLI for Windows | Microsoft Learn2](/cli/azure/).
2. **Execute the CreateDevAADApp.ps1 Script**: Execute the [CreateDevAADApp script](https://github.com/microsoft/Microsoft-Fabric-workload-development-sample/blob/main/Authentication/CreateDevAADApp.ps1). You're prompted to sign in by using the credentials of the user account under which you intend to create the application.
3. **Provide Required Information**: When prompted, enter the desired name for your application, the workload name (prefixed with "Org."), and your tenant ID.

Upon successful execution of the script, it outputs all necessary details to configure your workload. Additionally, it provides a direct URL to your application and an administrative consent URL for tenant-wide application authorization.

### Example of usage

To create an application named "myWorkloadApp" with the workload name "Org.Myworkload" for the specified tenant, run the following command in PowerShell:

```powershell
powershell .\CreateDevAADApp.ps1 -applicationName "myWorkloadApp" -workloadName "Org.Myworkload" -tenantId "cb1f79ad-7930-43c0-8d84-c1bf8d15c8c7"
```

This example demonstrates how to use the `CreateDevAADApp.ps1` script with command-line arguments to automate the application setup process. The provided tenant ID is an example and should be replaced with your actual tenant ID.

## Configuring your workload (Backend)

1. In the Backend sample, go to `src/appsettings.json` file in the [repository](https://go.microsoft.com/fwlink/?linkid=2272254) and configure the settings:

   * `PublisherTenantId`: The tenant ID of the publisher  
   * `ClientId`: Your application ID (you can find it in Microsoft Entra ID under overview).  
   * `ClientSecret`: The [secret you created](#generate-a-secret-for-your-application) when configuring the Microsoft Entra app.  
   * `Audience`: The [ID URI we configured](#configure-your-application-in-microsoft-entra-id-manually) in the Microsoft Entra app.  

1. Configure your *workloadManifest.xml*. Go to `src/Packages/manifest/files/WorkloadManifest.xml` file in the [repository](https://go.microsoft.com/fwlink/?linkid=2272254) and configure your `AppId`, `redirectUri`, and `ResourceId` (ID URI) under **AADApps**.
```
<AADApp>
    <AppId>YourApplicationId</AppId>
    <RedirectUri>YourRedirectUri</RedirectUri>
    <ResourceId>YourResourceId</ResourceId>
</AADApp>
```
## Configure the workload local manifest
> [!NOTE]
> This step is only applicable to the devmode scenario.

After configuring your application, update the following configurations in `.env.dev` configuration file located in the [Frontend folder](https://github.com/microsoft/Microsoft-Fabric-workload-development-sample/tree/main/Frontend):

```
"DEV_AAD_CONFIG_AUDIENCE": "", // The ID URI configured in your application for developer scenario

"DEV_AAD_CONFIG_REDIRECT_URI": "http://localhost:60006/close", // or the path you configured in index.ts

"DEV_AAD_CONFIG_APPID": "" // your app Id
```

:::image type="content" source="./media/authentication-tutorial/configure-workload-env-dev.png" alt-text="Screenshot that shows the configuration of a `.env.dev` file.":::

To understand how to work with authentication in workloads, see [Workload authentication guidelines & deep dive](./authentication-guidelines.md).
