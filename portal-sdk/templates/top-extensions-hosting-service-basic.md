

## How the hosting service serves an extension

 The runtime component of the hosting service is hosted inside an Azure Cloud Service. The extension developer provides a publicly accessible endpoint that contains the contents that the hosting service will serve. When an extension onboards to the service, the service  locates a file named  `config.json` in this endpoint.

The hosting service will upload the config file, and look into it to figure out which zip files it needs. There can be multiple versions of the extension referenced in `config.json`. The hosting service will upload them and unpack them on the local disk. After it has successfully uploaded and expanded all versions of the extension referenced in `config.json`, it will write `config.json` to disk.

For performance reasons, a version of an extension can only be uploaded once.

## Reasons for using the hosting service

More than 50% of the extensions have been migrated from legacy DIY deployment to extension hosting services. Some reasons for using the hosting service are as follows.

1. Simple deployments and hosting out of the box 
    
    * Use safe deployment practices

    * [Geodistributes](portalfx-extensions-hosting-service-glossary.md) the extension to all data centers

    * CDN configured

    * Use the Portal's MDS so there is no need to onboard to MDS

    * Use optimizations like persistent caching, index page caching, manifest caching and others 

1. Enhanced monitoring 

    * Removes the need for on-call rotation for hosting specific issues because the Portal is now hosting. On-call support is still required for dev code live site issues

    * Provides full visibility into the health and activity for an extension

1. Reduced COGS

    *  No hosting [COGS](portalfx-extensions-hosting-service-glossary.md)

    *  Reduced development costs allow teams to focus on building the domain specific portions of the extension, instead of allocating resources to configuring deployment


## Hosting services and server-side code

Extensions that have server-side code or controllers can use hosting services.  In fact, you can supplement a legacy DIY deployment infrastructure to use a hosting service, and deploy extensions in a way that complies with safe-deployment practices. 
1.	In most cases, UI controllers or [MVC](portalfx-extensions-hosting-service-glossary.md) controllers are legacy, and it is easy to obsolete these controllers. One advantage of replacing obsolete UI controllers is that all client applications, such as **Ibiza** and **PowerShell**, will have a consistent experience. You can replace UI controllers under the following conditions.
    *	If the functionality is already available from another service
    *	By hosting server-side code within an existing RP
1.	If replacing UI controllers is not a short-term task, the extension can be deployed through a hosting service by modifying the relative controller URLs.  They are located in  client code, and can be changed to specify absolute URLS. 

    <!--TODO: Locate a better word than "pull-request" for sample code.  This links to the commit branch instead of a request that can be sent to another team for processing. -->

    The following is a sample pull-request for a cloud services extension. [https://msazure.visualstudio.com/One/_git/AzureUX-CloudServices/commit/ac183c0ec197de7c7fd3e1eee1f7b41eb5f2dc8b](https://msazure.visualstudio.com/One/_git/AzureUX-CloudServices/commit/ac183c0ec197de7c7fd3e1eee1f7b41eb5f2dc8b).
    
    When this code change is posted, the extension can be deployed as a server-only service that is behind **Traffic Manager**.
    
    The extension versions that are available in the Hosting Service are located at the following URLs.

    * Dogfood: [https://hosting.onecloud.azure-test.net/api/diagnostics](https://hosting.onecloud.azure-test.net/api/diagnostics)
    * MPAC: [https://hosting-ms.portal.azure.net/api/diagnostics](https://hosting-ms.portal.azure.net/api/diagnostics)
    * PROD: [https://hosting.portal.azure.net/api/diagnostics](https://hosting.portal.azure.net/api/diagnostics)

 ## Prerequisites for onboarding hosting service

The **Visual Studio** project that is associated with developing the extension contains several files that will be updated or overridden while getting the extension ready for the hosting service. This topic discusses the files to create or change to meet requirements for the onboarding process. This procedure uses the **Content Unbundler** tool that was installed with the **nuGet** packages in Visual Studio.  For more information, see [portalfx-extensions-create-blank-procedure.md](portalfx-extensions-create-blank-procedure.md) and [top-extensions-nuget.md](top-extensions-nuget.md).

1. For all extensions
   * SDK Version 

     Use Portal SDK 5.0.302.454 or newer to generate the zip file during the extension build process.
    
     **NOTE**: If your team plans to use EV2 for uploading the zip file to its storage account, we recommend using Portal SDK 5.0.302.817 or newer. Some new features have recently been added that make it easier to use EV2 with a hosting service.

    * Build Output Format
      * Verify that the build output directory is named `bin`
      * Verify that **IIS** can point to the `bin` directory 
      * Verify that **IIS** will load extension from the `bin` directory

1. For extensions with controllers or server-side code

   Modify the relative controller URLs to contain absolute URLS. The controllers will deploy a new server-only service that will be behind the **Traffic Manager**.
   
   Because this process is typically the same across all extensions, you can use the following pull request for a cloud services extension.
  [https://msazure.visualstudio.com/One/_git/AzureUX-CloudServices/commit/ac183c0ec197de7c7fd3e1eee1f7b41eb5f2dc8b](https://msazure.visualstudio.com/One/_git/AzureUX-CloudServices/commit/ac183c0ec197de7c7fd3e1eee1f7b41eb5f2dc8b).

### Monitoring and Logging

#### Logging

  The Portal provides a way for extensions to log to MDS using a feature that can be enabled in the extension. The logs generated by the extension when this feature is enabled are located in tables in the Portal's MDS account. For more information about the Portal logging feature, see  [portalfx-telemetry.md#logging](portalfx-telemetry.md#logging).

#### Trace Events

Trace events are stored in a **po** database, and can be analyzed with the Kusto.WebExplorer tool. The following link contains a query that specifies which trace events to consider for analysis.
[https://ailoganalyticsportal-privatecluster.cloudapp.net/clusters/Azportal/databases/AzurePortal?query=ExtEvents%7Cwhere+PreciseTimeStamp%3Eago(10m)](https://ailoganalyticsportal-privatecluster.cloudapp.net/clusters/Azportal/databases/AzurePortal?query=ExtEvents%7Cwhere+PreciseTimeStamp%3Eago(10m))

The following image contains a list of tables that are a part of the Kusto database schema. It also displays the columns from the `ExtEvents` table that was used in the previous query.

 ![alt-text](../media/portalfx-extensions-hosting-service/KustoWebExplorerQuery.png  "Trace Event Parameters")

For more information about Kusto WebExplorer and associated functions, see [https://kusto.azurewebsites.net/docs/queryLanguage/query_language_syntax.html?q=cross](https://kusto.azurewebsites.net/docs/queryLanguage/query_language_syntax.html?q=cross). For questions to the developer community, use the  Stackoverflow tag [ibiza-deployment](https://stackoverflow.microsoft.com/questions/tagged/ibiza-deployment) and [ibiza-hosting-service](https://stackoverflow.microsoft.com/questions/tagged/ibiza-hosting-service).


#### Telemetry Events

Telemetry events are stored in the **Kusto** database in the `ExtTelemetry` table.  To  review telemetry events, use the following query against the Kusto.WebExplorer site.

[https://ailoganalyticsportal-privatecluster.cloudapp.net/clusters/Azportal/databases/AzurePortal?query=ExtTelemetry%7Cwhere+PreciseTimeStamp%3Eago(10m)](https://ailoganalyticsportal-privatecluster.cloudapp.net/clusters/Azportal/databases/AzurePortal?query=ExtTelemetry%7Cwhere+PreciseTimeStamp%3Eago(10m))

#### Monitoring

 There are two categories of issues that are monitored for each extension.

1.  Portal loading and running the extension

    The Portal has alerts that notify extensions when they fail to load for any reason. Issues like blade load failures and part load failures are also monitored.

    <!-- TODO: Determine whether the work that needed to be done to monitor blade load failures and part load failures has been done. -->
    <!-- TODO: Determine whether it is the extension that is notified, or the partner that is notified. -->

1. Hosting Service downloading and servicing the extension

    The hosting service pings the endpoint that contains the extension bits once every 5 minutes. It will then download any new configurations and versions that it finds. If it fails to download or process the downloaded files, it logs these as errors in its own MDS tables.

    Alerts and monitors have been developed for any issues. The Ibiza team receives notifications if any errors or warnings are generated by the hosting service. 
    You can access the logs of the hosting service at [https://jarvis-west.dc.ad.msft.net/53731DA4](https://jarvis-west.dc.ad.msft.net/53731DA4).

 