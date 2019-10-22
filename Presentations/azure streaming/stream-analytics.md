For our demo, we'll need a source of live data. This could be an event hub, an IOT hub or a  blob store. Here we'll use an Event Hub.  We'll monitor the data as it arrives in real time and process it using an Azure stream analytics query. Then we will take the results of that processing and send it to an output sync, which could be  database or  an blob store, or it could be a real time Power BI dashboard. For our demo we will use a sql server database

Lets provision a new event hub from the Azure Market place

From the navigation on the left hand side we click on create a resource

We can now Search for event hub in the search box

We'll  call it mywonderfuleventhub

We'll  use the standard pricing tier

Lets create a new resource group

In my region, which is the   central us region





Run Script

Now as we can see we have a Event hub, a SQL Server and a database 

(click on SQL Server, then sql databases) called devicedb

(click on devicedb, then query editor at top, under overview)

Lets log into the query editor and create a table

CREATE TABLE [dbo].[AuditTable](
    [Id] [int] IDENTITY(1,1) NOT NULL,
    [device] [varchar](50) NULL,
    [tenant] [varchar](50) NULL,
    project [varchar](50) NULL,
    [username] [varchar](50) NULL,
    [action] [varchar](150) NULL,
    [tshirtcolor] [varchar](150) NULL,
    [EventEnqueuedUtcTime] [datetime] NULL
) ON [PRIMARY]

(expand tables and dbo.audittable)

Great we have a table called AuditTable where Azure Stream Analytics will pump data.¬