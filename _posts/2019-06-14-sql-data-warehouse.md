---
layout: post
title: "Azure SQL Data Warehouse"
author: "Hersh Bhasin"
comments: true
categories: Azure-Sql-Data-Warehouse Big-Data AZ-300
---

![](../assets/dw1.png)

![](../assets/dw2.png)

Above pic. shows that each compute node has a db (a distribution). A control node controlls  all compute nodes. Storage and compute can be scaled independently.

# Distribution Methods and Keys

Azure SQL Data Warehouse uses  60 distributions (buckets) when loading data into the system. Distribution can be based on:

1. Hash Distribution: (say distribution by a Product). A good has should be at least 60 different types (to match 60 distributions)
2. Round Robin Distribution

![](../assets/dw3.png)

Each distribution is a sql server database. (1 distribution = 1 sql server db). Each SQL Server Datawarehouse splits into 60 of these distributions.

Compute is increased (scale-out) by increasing DWU units in multiples of 100. Each DWU100 increment means adding 1 additional compute node.

**Example of compute and distribution relationship**:

DWU100 = 1 compute node.  Hence all 60 distributions (dbs) will reside on a single compute node.

DWU200 = 2 compute nodes (pic above). Hence 30 distributions will reside on each compute node.

And so on ...

# Table Type

* Clustered Columnstore: for large tables, organized by columns. Default table type. High compression ratio. Ideally segments of 1M rows. No secondary indexes.
* Heap: meant for temp or smaller tables. No native index, no ordering. Fast load. No compression. Allows secondary indexes.
* Clustered B-Tree Index: most common type, found on on-premise SQL Server. Sorted index on data. Fast singleton lookup . No compression. Allows secondary indexing.

![](../assets/dw4.png)

# Distributions

![](../assets/dw5.png)

![](../assets/dw7.png)

**Hash Distribution**

Hash is a function  where we feed in a number and then it produces an output called the Hash Value. So what's going to happen is when a row gets inserted into the table using that table definition that we created on the far right screen, the value of the column is going to get passed into a Hash function and the Hash value that gets passed returned out is either going to be the same value each time, meaning that it's deterministic or it's going to be non-deterministic, meaning you pass in the same value and you get a different value. *In SQL Data Warehouse,we always have the same value generated. So it's a deterministic Hash function.*

Hash(x) = y

![](../assets/dw 2.png)

One of the things that is important when you're deciding on a column to Hash
on, is that you select a column that's going to give you an even
distribution of data across those 60 distributions. And this is because the
distributions are assigned to compute nodes and you want to have that
parallel work done in equal chunks, so that each chunk of work that's done
completes at roughly the same time.

You want to avoid hashing on a column that has a lot of null values, because
they will always go into the same bucket and you won't get that even
distribution of data that helps reduce data movement.

**Round Robin Distribution**

Our other distribution option is Round Robin and this is the default
distribution method, if you don't specify one when you creating your table.
The Round Robin distribution method is fairly simple to set up, but because
we're not being strategic about the location of the data across the
distributions, we may end up moving data at query time and data movement is
expensive and it can affect our performance.

**Optimizing Distribution**

![](../assets/dw 8.png)

1. Optomize For Data Movement
   The most optimized queries on an MPP system can
   just be passed through to an individual distribution or distribution
   database and be executed just in that distribution on their own without
   having to pull data, move data, from other distributions to solve a query.
   That's ideal, we can't always avoid moving data, but when we can make a
   choice about our distribution that affects data movement we want to do that.

2. Optomize For Data Skew
   We also want to minimize data skew and data skew happens when data
   distributions have a disproportionate amount of data relative to the other
   distribution. 

**Movement of Data**

Okay, let's say we have two tables and we distribute them. 

* Scenario 1 : both tables hashed on order id = > (no data movement)

  We distribute one table on order ID, and the other table gets distributed an order ID. Because our hash function is deterministic, we pass in an order ID from table one that gets placed in bucket X. We pass in the product ID to the hash for table two, it's still order ID, that's still going to go in the same bucket. So, then all those same order IDs are in the same distribution or bucket, and when a query is issued that has to look for a specific order ID and it joins like table A table B, Bingo! All that data is in one distribution. So, we don't have to go somewhere else to get more information. 

* Scenario 2:  Table A  is distributed on order ID, table B is just distributed on date =>(data movement)

   So then let's think about another scenario where we have a different distribution for these two tables. So, table A is distributed on order ID, table B is just distributed on date. Now, they will have different hash values, that means they go in different buckets. So, your product IDs, go in a certain bucket, your order ID is going a certain bucket. If you need to join these two tables, maybe you wanted to look at a specific customer ID and  want to look at the orders they placed on a certain date. Now you won't have all the data you need for that customer placed in a single distribution. So, you've got to move data between the distributions in order to or between the compute nodes in order to satisfy that. 

  So, the downside of data movement is that it's slower and it's going to take more resources as well more memory, more compute. 

# Example Round Robin

```sql
IF OBJECT_ID('dbo.OrdersRR', 'U') IS NOT NULL 
   DROP TABLE dbo.OrdersRR;
GO 

CREATE TABLE OrdersRR (
  OrderID int IDENTITY(1, 1) NOT NULL, 
  OrderDate datetime NOT NULL, 
  OrderDescription char(15) DEFAULT ' NewOrdersRR'
) WITH ( CLUSTERED INDEX (OrderID), DISTRIBUTION = ROUND_ROBIN );
```

Code to enter rows of data into the newly created table:

```sql
SET NOCOUNT ON 

DECLARE @i INT 
SET @i = 1 
DECLARE @date DATETIME 
SET @date = dateadd(mi, @i, '2018-01-29') 

WHILE (@i <= 60) 
BEGIN 
   INSERT INTO [OrdersRR] (OrderDate) 
   SELECT @date 

   SET @i = @i + 1;
END		
```

Query which makes use of the SQL DW DMVs to show how data was distributed across the distributions using the round robin distribution method.

```sql
SELECT 
  o.name AS tableName, 
  pnp.pdw_node_id, 
  pnp.distribution_id, 
  pnp.rows 
FROM 
  sys.pdw_nodes_partitions AS pnp 
  JOIN sys.pdw_nodes_tables AS NTables ON pnp.object_id = NTables.object_id 
     AND pnp.pdw_node_id = NTables.pdw_node_id 
  JOIN sys.pdw_table_mappings AS TMap ON NTables.name = TMap.physical_name 
     AND substring(TMap.physical_name, 40, 10) = pnp.distribution_id 
  JOIN sys.objects AS o ON TMap.object_id = o.object_id 
WHERE 
  o.name in ('orders') 
ORDER BY 
  distribution_id			
```

In the results, I can see that the data was distributed somewhat evenly across the 60 distributions.

![](../assets/dw 9.png)

# Example Hash

Data for the hash distributed table gets distributed across multiple distributions and eventually gets processed by multiple compute nodes in parallel across all the compute nodes. Fact tables or large tables are good candidates for hash distributed tables. You select one of the columns from the table to use as the distribution key column when creating a hash distributed table and then SQL Data Warehouse automatically distributes the rows across all 60 distributions based on distribution key column value.

```sql
CREATE TABLE OrdersH (
  OrderID int IDENTITY(1, 1) NOT NULL, 
  OrderDate datetime NOT NULL, 
  OrderDescription char(15) DEFAULT 'NewOrdersH'
) WITH ( CLUSTERED INDEX (OrderID), DISTRIBUTION = HASH(OrderDate) );
```

Enter rows of data into the newly created table:

```sql
SET NOCOUNT ON 

DECLARE @i INT 
SET @i = 1 
DECLARE @date DATETIME 
SET @date = dateadd(mi, @i, '2017-02-04') 

WHILE (@i <= 60) 
BEGIN 
   INSERT INTO [Orders2] (OrderDate) 
   SELECT @date 

   SET @i = @i + 1;
END 
```

Query which makes use of the SQL DW DMVs to show how data was distributed across the distributions using the hash distribution method.

```sql
SELECT 
  o.name AS tableName, 
  pnp.pdw_node_id, 
  pnp.distribution_id, 
  pnp.rows 
FROM 
  sys.pdw_nodes_partitions AS pnp 
  JOIN sys.pdw_nodes_tables AS NTables ON pnp.object_id = NTables.object_id 
  AND pnp.pdw_node_id = NTables.pdw_node_id 
  JOIN sys.pdw_table_mappings AS TMap ON NTables.name = TMap.physical_name 
  AND substring(TMap.physical_name, 40, 10) = pnp.distribution_id 
  JOIN sys.objects AS o ON TMap.object_id = o.object_id 
WHERE 
  o.name in ('OrdersH')			
```

The query results show that all the rows went into the same distribution because the same date was used for each row and the SQL Data Warehouse hash function is deterministic.

![](../assets/dw 10.png)



# Relicated Tables

Replicated tables are a tool for avoiding data movement.

Lets review what data movement is first. So the most optimized queries in an MPP system can simply be passed through to execute on individual distributed databases with no interaction between the other databases. But sometimes, data movement is needed between distributions to satisfy the join criteria in a query. 

Replicated tables are a tool that we can use to avoid data movement for small tables. The replicated table has a full copy that's accessible by each compute node. And replicating a table removes the need to transfer data among the compute nodes before a join or an aggregation.

Replicated tables should be small < 2 gas. Replicated tables typically work well for small tables that are dimension tables in a star schema. A dimension table is is something that stores descriptive data that typically changes slowly, so things like customer name, address, or product key.

![](../assets/dw 3.png)

```sql
CREATE TABLE [dbo].[DimSalesTerritory_REPLICATED] 
WITH ( CLUSTERED COLUMNSTORE INDEX, 
         DISTRIBUTION = REPLICATE ) 
AS 
SELECT * FROM [dbo].[DimSalesTerritory_RR] OPTION (LABEL = 'CTAS : DimSalesTerritory_REPLICATED')
```

Create Statistics

![](../assets/dw 4.png)

Use the sys.dm_pdw_exec_requests DMV to see query plan containing information about all requests currently or recently active in SQL DW, and the sys.dm_pdw_request_steps DMV, which contains information about all the steps that make up a given request or query.

```sql
SELECT 
   step_index, 
   operation_type
FROM 
   sys.dm_pdw_exec_requests er
   JOIN sys.dm_pdw_request_steps rs ON er.request_id = rs.request_id
WHERE 
   er.[label] = 'STATEMENT:RRTableQuery';
```

**BrodcastMoveOperation**

A BrodcastMoveOperation indicates data movement. And this type of data movement operation will slow down query performance and it can be eliminated by using Replicated Tables.

We will still see  BrodcastMoveOperation the first time that we run a query after we've replicated a table. And this is important to note so that when you do this you don't think something's gone wrong. This is normal. When you replicate a table the first time, it's still going to stay that Round Robin Table. But the process of issuing a query that needs to use that Replicated Table will actually replicate that table across the compute nodes, the first time the query's executed. So this is why we see the Broadcast Move Operation happening the first time but we're not going to see it the second time.

#  Table  Partitioning

In data warehousing, we have to run load processes to put data into the warehouse, and we want those processes to run as efficiently as possible. Table partitioning is a tool that helps us with that efficiency.  Partitioning helps us divide a large table into smaller parts, and those small parts give us the benefits of faster data loading and faster querying. We have the opportunity with partitioning to switch data in and out of our tables to help with our load process. The partitioning is not visible to our end users, so when they query the database, it just appears like it's one logical structure.

![](../assets/dw 5.png)

**Partition Switching , Elimination and Partition Merging**

Some of the benefits that you get in your load process come from this partition switching and partition merging. The main benefit is that a partition switch or merge into an empty table or an empty partition is simply a metadata process. *It's actually not a logged transactio*n. And so, by avoiding transaction logging, you have some benefits over simply deleting data at the table. 

So let's consider an example where you have a table that contains 36 partitions, and you're partitioned on month. So, at the end of 36 months, on your 37th month, you have to archive out to your oldest partition. And so you would have two options in doing this. If you had a non-partition table what you might do is delete that data. So you'd say, 

>  "Delete from table where month equals last 36th oldest month." 

And what would happen is, row by row, the data would be deleted. *It would be fully-logged operation*, and it would also potentially take a long time to rollback the transaction if something went wrong. So, we've got a lot of I/O happening from the log writes. We have a row-by-row operation, which is not optimal for SQL Data Warehouse. 

And then if we compare this to the partition switch, if we just had a partition table, and we were to partition switch the 36th month out of the table, then what happens is that we have an instantaneous operation that isn't logged, and we have some manageability benefits as well.

We also get some benefits to queries. With queries, there's a concept called *partition elimination* that we could take advantage of. 

So, when you have a column listed in your WHERE clause in your query, and it is filtering your data on the same column that you've partitioned your table, what that allows SQL Data Warehouse to do is to not look at the partitions that don't meet that criteria that you're filtering on. That's partition elimination, and it gives us the benefit of having to scan less data and to return results more quickly.

![](../assets/dw 6.png)

![](../assets/dw 11.png)

So, if we only look the boundary ranges, let's determine what that means first. So, we start with 2017, and we've got February 5, 2017, then we have February 12, 2017. These are all Sundays. *So I'm partitioning basically on week*. Two 26, three five, three 12, three 19. 

So what does that mean? Well, when we're talking about partitions, we're talking about the boundary or the range of values that fall between for example, February 5th and February 12th of 2017. So, when we have a range right partition, we automatically get a partition to the left of the first values specified. So, any dates that were OrderDates that were inserted into this table that were less than this first boundary value, February 5th of 2017, they'd fall into this first bucket that was first partition. So that would be identified as partition one in metadata in C2 DW. Then our next partition boundary would be between the first and second OrderDate values that we specified. So between anything that's greater than or equal to February 5th of 2017 but less than February 12th of 2017, et cetera as we go down to our last partition. Our last partition is three 19, March 19th of 2017. Anything greater than or equal to that value falls into the final partition. Now, if I count these up we have one, two, three, four, five, six, seven boundary values we specified.  We get the leftmost partition, I guess you could say for free. And so that's identified as partition one, so we get to take the number of boundary values plus one. So N plus one is a number of partitions. We have eight of them. So one, two, three, four, five, six, seven, eight. I'm enumerating all these details because we're going to talk about what falls into which partition as we answer to values. 



![](../assets/dw 7.png)

60 Distributions

  Each Distribution

​	Has 4 Partitions

​		Each Partition

​			Has Row Groups (1 million rows per distribution and partition). Need at least 60 million rows 			per partition.

So all we do is we specify the partition values and the boundary values and we instantly have our partitions created for our table.

And this is pretty neat as you compare it to SQL Database on-premise because you don't have any partition functions or partition schemes that you have to define separately from the CREATE TABLE statement. So all we do is we specify the partition values and the boundary values and we instantly have our partitions created for our table.

So, if we want to think about what happens when we load data to this table, it's important to think about this because we have distributions, we have clustered columnstore, and we have partitions. 

So, what's going to happen here is we have *60 distributions*, like we've talked about previously in our distribution module. So we have 60 buckets of data, 60 separate Azure SQL Databases. So then our data gets each row that has to get inserted. It gets inserted into one of those buckets. Then within the bucket, we have *five partitions*. For this partition definition, we have four partitions defined and then we have a leftmost partition. That's technically not shown on the slide but we get it by default, so n plus one partitions. 

So that row comes in, it goes into a distribution, and then within each distribution, there are five partitions. And then within each partition, there are *row groups*. A row group is a component of clustered columnstore, and we'll talk more about that in our columnstore module. But we have this hierarchy. 

So what happens is we end up with a distribution, then we have a partition, and then we have a clustered columnstore row group. One of the things that happens is that we have to take advantage of clustered columnstore by grouping our rows into *1 million row groups*. So, within this hierarchy, we need to make sure for partitioning a table that we have at least 1 million rows that meet each partition boundary value that we're setting up. So, we need at least 60 million rows per partition so that we have minimum of 1 million rows per distribution. 

So, if we had the 36-month example, we would need to have 1 million rows per month times 60 million rows per partition, times 1 million rows per row group. And then that comes to 2.1 billion rows. *So when you apply partitioning to a table that has cluster columnstore, you really need to make sure that you're going to have at least 1 million rows per distribution and partition to take advantage of the compression and performance benefits that you get from cluster columnstore.*

#References