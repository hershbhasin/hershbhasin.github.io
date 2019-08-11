---
layout: post
title: "Embedded Reporting with Azure Power BI Embedded"
author: "Hersh Bhasin"
comments: true
categories: Big-Data
---
## Introduction 

Power BI Embedded is intended to simplify how ISVs and developers use Power BI capabilities with embedded analytics. In this proof of concept, I will show you how to set up an Azure  Power BI Embedded environment. In an earlier POC: [Event Streaming using Azure Stream Analytics](https://hershbhasin.com/2019-08-05/azure-stream-analytics), I demonstrated the use of Azure Stream Analytics to stream event hub data into a Azure SQL Server database. This POC extends the earlier streaming POC and shows how to build reports off the event hub data, how to host the reports on Azure using PowerBI embedded and finally, how to embed the reports in a custom MVC application.

This is a two part post:

1. [Azure Stream Analytics](https://hershbhasin.com/2019-08-05/azure-stream-analytics )
2. [Embedded Reporting with Azure Power BI Embedded](https://hershbhasin.com/2019-08-06/powerbi-embedded)


## Solution Overview

![](/assets/powerbi_1.PNG)

## Report Creation

1. Report should be created in Power BI desktop ([Free Download)](https://powerbi.microsoft.com/en-us/desktop/)
2. Data Connectivity mode: should be DirectQuery (so data is always current. We can also explore schedule refreshes if DirectQuery is considered sub optimal)
3. Azure BI Embedded currently supports only SQL Server & SQL Server  warehouse as data sources (? to confirm)
4. A PowerBI report is provided under PowerBI/Reports/AuditLog.pbix that connects to the Azure SQL Server database created earlier (devicedb ) in post: [Event Streaming using Azure Stream Analytics](https://tech.hershbhasin.com/2019/04/event-streaming-using-azure-stream_2.html)

## Hosting with Power BI Embedded

- Azure AD used to authenticate users and apps
  - PBI licenses are assigned to Azure AD user accounts
  - Organization owns a tenant (i.e. directory)
  - AAD tenant contains user accounts and groups
  - AAD tenant contains set of registered applications
- You must register your application with Azure AD
  - Requirement of calling to Power BI service API
  - Applications registered as Web app or Native app
  - Registered applications are assigned GUID for client ID
  - Application is configured with permissions

1. Follow this quickstart to set up the PowerBI Hosting environment  & to create a workspace: <https://docs.microsoft.com/en-us/power-bi/developer/embed-sample-for-customers>
2. You will need to create an account on [https://app.powerbi.com](https://app.powerbi.com/groups/me/list/dashboards).
3. Now, using PowerBI desktop, we can publish the report AuditLog.pbix to the workspace from the Publish tab in PowerBI. The report gets published to your account at powerbi.com
4. Once published, log into [https://app.powerbi.com](https://app.powerbi.com/groups/me/list/dashboards) and under workspace/Datasets/settings/Data source credentials, set your Azure SQL Server credentials  (Basic Authentication Method)

## Embedding a Power BI Report in an Application

There is a sample MVC application provided in the PowerBI/App Owns Data folder that will run our PowerBI Report

The following items in Web.config need to be filled. Refer to <https://docs.microsoft.com/en-us/power-bi/developer/embed-sample-for-customers> (section: Embed your content using the sample application) to fill in values for these 5 items

```javascript
<add key="applicationId" value="a9dd54e0-7f89-4056-9ce4-4d84d0bf1f1b" />
<add key="workspaceId" value="252c3628-aab8-4fec-a3da-bbe6f3924905" />
<add key="reportId" value="f90ff3b3-fe68-43e1-852d-29d9049dca17" />
<add key="pbiUsername" value="xxxxxx" />
<add key="pbiPassword" value="xxxxxx" />
```

The report runs within the MVC application on Localhost

![](/assets/powerbi_2.PNG)

## Source Code Download

[Source Code](https://github.com/hershbhasin/AzureSamples/tree/master/AzureStreaming)

## References

 [Microsoft Power BI Embedded Playground](https://microsoft.github.io/PowerBI-JavaScript/demo/v2-demo/index.html)

[Row Level Security](https://docs.microsoft.com/en-us/power-bi/developer/embedded-row-level-security)


