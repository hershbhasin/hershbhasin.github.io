---
layout: post
title: "Optimizing Azure Data Lake"
author: "Hersh Bhasin"
comments: true
categories: Azure-Data-Lake
published: true
---



# Partitioning Vs Distribution

**Partitioning**: How data is stored

1. Horizontal Partitioning (Distributed By). Aka Fine Grained
2. Vertical Partitioning (Partitioned By).  Aka Corse grained. Developer controls sets up Vertical Partitions (mostly used for date. Say partition my Year Month. Developer has to create these partition buckets). Within Vertical partition, you can have horizontal partitions.

**Distribution**: Where data is stored. Used in conjunction with Horizontal Partitioning. Data can be distributed by

1. Hash
2. Direct Hash
3. Range (of keys)
4. Round Robin

Hash

```sql
USE DATABASE UkPostcodes;
USE SCHEMA PoliceData;

DROP TABLE IF EXISTS StreetCrime;

CREATE TABLE IF NOT EXISTS StreetCrime
(
    CrimeID string,
    Outcome string,
    CurrentFileDate DateTime,
    INDEX idx_StreetCrime
    CLUSTERED(ShortReportedByPoliceForceName)
    DISTRIBUTED BY
    HASH(ShortReportedByPoliceForceName)
);
```



**Partitioned By**

![](..\assets\dl1.PNG)