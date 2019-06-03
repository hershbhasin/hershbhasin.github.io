---

layout: post

title: "Live Streaming with Azure Stream Analytics"

author: "Hersh Bhasin"
---

Azure Stream Analytics is a Cloud service in Azure that is used to process real time streams of data. The input for live data can be Azure event hub, an IOT hub or an Azure blob store location. As the data arrives in real time, we process it and send the results of that processing to an output sync which could be could be an Azure blob store location, or an Azure data location.

To process the data between the input & output , we define a query. The query is written using standard SQL syntax and we run that query on the data. Once we start our Azure Stream Analytics query, as that data comes in in real time, the query will run against the unbounded stream of data as it arrives and write the results to the output. Various rules can be applied to the data and these rules can be applied before the data is persisted to an output. For example threshold rules can be applied so that only events exceeding certain threshold limits are persisted.

In this POC, we will take a stream of live events entering a Event Hub and send the results to a Azure SQL Server database. In a later POC, we will use this database for reporting purposes.

## Create Azure Environment

In the source, there is a powershell script called reportingpoc.ps1. Run this script to set up the environment in Azure. A Event Hub namespace and a SQL  Server database is created.

![](/assets/streaming_1.png)

## Create Database table

In the Azure SQL Database (devicedb), create the following table

```sql
CREATE TABLE [dbo].[AuditTable]

( [Id] [int] IDENTITY(1,1) NOT NULL,

 [device] varchar NULL,

 [tenant] varchar NULL,

 project varchar NULL,

 [username] varchar NULL,

 [action] varchar NULL,

 [EventEnqueuedUtcTime] [datetime] NULL )

 ON [PRIMARY]

```

### Create a Stream Analytics Job

1. Add a Stream Analytics Job (from Marketplace)

2. Create a input (Select the Event Hub)

   ![](/assets/streaming_2.png)

3. Create an Output by pointing to the SQL Server database

   ![](/assets/streaming_3.png)

4. Create a SQL Query

   ```sql
   SELECT device,tenant,project,username,action,EventEnqueuedUtcTime 
   INTO output 
   FROM input
   ```

   

   ![](/assets/streaming_4.png)

5. Start the Job

### Send Events to the Event Hub

The source code has a console application called "Sample Sender". After adding the connectionstring for the EventHub in the property EventHubConnectionString (in program.cs), run the application. The application will write 200 random events  as per this sample:

```json
{"device":"Acendo Core 456","username":"Sue","action":"ABC Integrators","project":"Richardson"} 
```



The Streaming Analytics job will insert these events in the SQL Server table created earlier

### Source Code

[Source Code](https://github.com/hershbhasin/AzureSamples/tree/master/AzureStreaming)

### References

[Stream Analytics Documentation](https://docs.microsoft.com/en-us/azure/stream-analytics/)