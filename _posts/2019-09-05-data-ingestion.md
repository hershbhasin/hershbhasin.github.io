---

layout: post
title: "A Cost Effective Data Ingestion & Curation Pipeline in Azure"
author: "Hersh Bhasin"
comments: true
categories: Big-Data
---

## Introduction

For analytics data, we would need to store large volumes of data. Storage in CosmosDB or SQL Server is very expensive (for example, in CosmosDB there is a storage charge and a RU charge per query). Storage in Data Lake is cheap, and for cost effectiveness, analytics data  can be persisted/archived in a Data Lake. Please refer to the price comparisons section below.

This is a two part post:

[Part 1: Data Ingestion](https://hershbhasin.com/2019-05-05/data-ingestion)

[Part 2: Data Curation with Azure Data Factory](https://hershbhasin.com/2019-06-02/azure-data-factory)

## Price Comparison between CosmosDb and Data Lake


Here are the price comparisons for storing 1TB of data in a Data Lake vs Cosmos Db. You will note from the following costing that Cosmos db is 88% more expensive than a Data Lake with Data Lake Analytics:

**Pricing per Microsoft Azure Calculator**

![](/assets/ingest_1.PNG)

**CosmosDB**

Per Month 1 TB Storage, 20,000 RU/Sec (1 year reserved. This is the minimum you can reserve on fixed monthly commitment package):  

![](/assets/ingest_2.PNG)

**Data Lake**

Per month: I TB Storage, 1,000,000 Reads, 1,000,000 writes.  With monthly commitment package, I TB  cost is only $35/Month

![](/assets/ingest_3.PNG)

*Conclusion: Cosmos db is 88% more expensive than a Data Lake with Data Lake Analytics*

# Justification for  Storing Raw Data in Data Lake

Storage in Data Lakes is cheap and is the preferred storage for Big Data like device event logs. Conceptually, a data lake is nothing more than a data repository. The data lake can store any type of data. Cost and effort are reduced because the data is stored in its original native format with no structure (schema) required of it initially. We can transform the data later as and when need arises. This is the "ELT" strategy in Big Data, where ELT stands for: Extract, Load and Transform. This strategy aligns with the "Schema on Read" technique, where we are able to store data relatively easily, with little upfront investment, then query the data "where it lives" without being required to relocate the data first. This technique  is different from the traditional "ETL" strategy where Transformation occurs before  Load to the data warehouse - this is referred to a "Schema on Write" because the schema, i.e., structure of the data, must be defined before the data is loaded to the data warehouse. This takes time and effort to do correctly. It also means we typically need to define use cases ahead of time in order to validate the schema is correct. 

 We would like to treat this data as immutable data because we might want to harvest this data for various purposes, and to go back to a point in time, if necessary. Also, data in Event Hubs are transient, typically held for a week. To persist this data, it is very easy to connect it to a Data Lake. The speed at which data arrives from an Event Hub is a non-issue for a Data Lake, and typically a data lake accommodates both a batch layer (data loaded from batch processes) and a speed layer (data loaded from streaming data or near real-time applications).

# Curating and Transforming Data

Data required for reporting purposes can be curated from the Data Lake. Such curated data can have a schema applied to it. For example data from Event Hubs is stored in the Data Lake with an Avero schema, and the Body is stored as  a byte array. We can transform this data, by applying a custom schema and persist the curated data as either csv files in the Data Lake or to a database table. We can also summarize and aggregate this data and store the summarized data in a curated folder in the Data lake

**Data archived in a Data lake can  be curated as follows:**

1. As a scheduled batch job: scheduled reports for a period can be made available as csv files (also residing in the Data Lake) and displayed as links to csv files in a  Web Report Area. 
2. As Batch Jobs: users will be able to create any type of report  based on the raw data persisted in the Data Lake. Since all raw data will be  persisted, this will include summarizing / aggregating reports etc. The user will submit his request and a background job will run, the output of which will be a CSV file. When the job is completed, user will get a notification that the report is available as a CSV file on the Report area.

# Proof of Concept

I will  now demonstrate how Event Hub data can be stored and curated from  a Data Lake.  In the source code provided, a Event Hub client application will write events to an event hub, The event hub will archive its contents into a Data Lake and a Data Lake Analytics job will curate this data as csv files.

![](/assets/ingest_4.PNG)

## Create Data Lake and Data Analytics
Create a Data Lake Store and a Data Analytics Service. This can be done by running the provided ARM template in source code folder: 

```
ARMTemplates\Data Ingestion\DataLake_DataAnalytics.json. 
```

You can run the template in the  Azure portal under *All Services/ Template (preview).* This will create a DataLake and a Data Analytics service

## Set Up Archive Folder in Data Lake

In this section, you create a folder within the account where you want to capture the data from Event Hubs.

You also assign permissions to Event Hubs so that it can write data into a Data Lake Storage Gen1 account.

Open the Data Lake Storage Gen1 account where you want to capture data from Event Hubs and then click on Data Explorer.

![](/assets/ingest_5.PNG)

Click New Folder and then enter a name for folder where you want to capture the data.

![](/assets/ingest_6.PNG)

**Assign permissions at the root of Data Lake Storage Gen1.**

a. Click Data Explorer, select the root of the Data Lake Storage Gen1 account, and then click Access.

![](/assets/ingest_7.PNG)

b. Under Access, click Add, click Select User or Group, and then search for Microsoft.EventHubs.

![](/assets/ingest_8.PNG)

Click Select.

c. Under Assign Permissions, click Select Permissions. Set Permissions to Execute. Set Add to to This folder and all children. Set Add as to An access permission entry and a default permission entry.

 Important

When creating a new folder hierarchy for capturing data received by Azure Event Hubs, this is an easy way to ensure access to the destination folder. However, adding permissions to all children of a top level folder with many child files and folders may take a long time. If your root folder contains a large number of files and folders, it may be faster to add Execute permissions for Microsoft.EventHubs individually to each folder in the path to your final destination folder.

![](/assets/ingest_9.PNG)

Click OK.

**Assign permissions for the folder under the Data Lake Storage Gen1 account where you want to capture data.**

a. Click Data Explorer, select the folder in the Data Lake Storage Gen1 account, and then click Access.

![](/assets/ingest_10.PNG)

b. Under Access, click Add, click Select User or Group, and then search for Microsoft.EventHubs.

![](/assets/ingest_12.PNG)

Click Select.

c. Under Assign Permissions, click Select Permissions. Set Permissions to Read, Write,and Execute. Set Add to to This folder and all children. Finally, set Add as to An access permission entry and a default permission entry.

![](/assets/ingest_13.PNG)

Click OK.

## Create an Event Hub and set it up to Archive to the Archive Folder set up above
In the source code folder, I have provided an ARM template called :

```
ARMTemplates\Data Ingestion\EventHub_DataLake.json.
```

You can run the template in the  Azure portal under *All Services/ Template (preview)*.

Running this template will create an Event Hub and set it up to capture Event Hub data to the archivefolder of the DataLake created earlier.

If you look at the Capture section of the eventhub, you will notice that: 

1. Capture has been turned on

2. Capture will happen every 5 min

3. Empty headers will not be written (otherwise you will have empty files created)

4. Event Hub points to the data lake created earlier

5. Points to the archivefolder

6. The Capture File format has been set to:

   ```json
   "{Namespace}_{EventHub}_{PartitionId}/{Year}/{Month}/{Day}_{Hour}_{Minute}_{Second}"
   ```

   The Data Lake will accumulate avro files by month. Look at the *data/archive* folder in the source code to see how the data will be stored in the data lake

7. Once created, Event Hub will save the events as avro files in Data Lake Store every 1 minute.

[](/assets/ingest_14.PNG)

# Sending data to Event Hub
A sample Event Hub events generator C# Console Application has been provided in the source code at: *..\EventHubClient\SampleSender*

1. Get the Event Hub connection string from Settings/Shared access policies
2. Update variable EventHubConnectionString in program.cs in provided sample: *SampleSender*
3. Run the sample app

**Sample Message Sent to Event Hub**

```json
{"timestamp":"2019-04-22T16:20:36.480717-05:00","device":"Room Book 1001","category":"Network","priority":"Warning","message":"Login Timeout","project":"Richardson","tenant":"ABC Integrators"}
```



# Transform  the Data


In the source code provided, open the solution *CloudworxUSQLApplication* in Visual Studio.

**Register assemblies**

Copy the following files from your CloudworxUSQLApplication/Lib directory to a directory in Azure Data Lake Store (e.g. \Assemblies\Avro):

1. Microsoft.Analytics.Samples.Formats.dll

2. Avro.dll

3. log4net.dll

4. Newtonsoft.Json.dll

   

**Open Project CloudworxUSQLApplication in Visual Studio and:**

1. Create a database ( run *1-CreateDB.usql.cs*), switch to the new database
2. Check file paths in *2-RegisterAssemblies.usql* and update them if necessary
3. register the assemblies which have previously been uploaded to ADLS by submitting *2-RegisterAssemblies.usql*
4. Run *logs.usql* job



# References

 [Microsoft Documentation](https://docs.microsoft.com/en-us/azure/data-lake-store/data-lake-store-archive-eventhub-capture)



## Source Code

The source code for this blog post is available here: [Source Code Link](https://github.com/hershbhasin/AzureSamples/tree/master/BigData)