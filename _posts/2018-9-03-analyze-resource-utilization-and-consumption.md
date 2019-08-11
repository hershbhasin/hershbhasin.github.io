---
layout: post
title: "Analyze resource utilization and consumption"
author: "Hersh Bhasin"
comments: true
categories: Azure-Monitoring AZ-300
published: true
---

![](..\assets\monitor1.PNG)

![](..\assets\monitor2.PNG)

[Enabling Diagnotic  Settings using CLI](http://techgenix.com/azure-diagnostic-settings/)

# Kusto Query Language (KQL)

[(KQL) syntax reference](https://docs.microsoft.com/en-us/sharepoint/dev/general-development/keyword-query-language-kql-syntax-reference)

[Pluralsight KQL Course](https://app.pluralsight.com/library/courses/kusto-query-language-kql-from-scratch/table-of-contents)

[Getting Started with the Kusto Query Language (KQL)](https://blogs.msdn.microsoft.com/ben/2018/10/11/getting-started-with-the-kusto-query-language/)

**KQL Playgrounds**

[Log Analytics Playground](https://portal.loganalytics.io/demo#/discover/query/main)

[Application Insights Playground](https://analytics.applicationinsights.io/demo#/discover/query/main)

[microsoft-defender-atp/advanced-hunting](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/advanced-hunting)

**Usage data volume per data type**

Where is your spend going?

```powershell
Usage 
| where TimeGenerated > ago(24h) and IsBillable == true and QuantityUnit == "MBytes"  //examine only billed per-MB usage data from the last 24 hours
| summarize VolumeMB = sum(Quantity) by DataType  //calculate total ingested MB per datatype 
| render piechart  //show results as pie chart
```

