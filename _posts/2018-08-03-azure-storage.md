---

layout: post
title: "Azure Storage"
author: "Hersh Bhasin"
comments: true
categories: Azure-Storage AZ-300
published: true
---

# Introduction

Study notes for [eDX course: Azure Storage](https://courses.edx.org/courses/course-v1:Microsoft+AZURE205x+1T2019a/) 

![](..\assets\storage1..PNG)

![](..\assets\storage2.PNG)

![](..\assets\storage3.PNG)

## General and Blob Storage Accounts

Standard & Premium

![](..\assets\storage4.PNG)

![](..\assets\storage5.PNG)



Azure Storage provides [three distinct account options](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-options), with different pricing and features supported. There are three kinds of storage account options: **General purpose v2 (GPv2)**, **General Purpose v1 (GPv1)**, and **Blob**.

A **General-purpose v2 (GPv2) storage account** is a storage account that supports all of the latest features for blobs, files, queues, and tables. GPv2 accounts support all APIs and features supported in GPv1 and Blob storage accounts. They also support the same durability, availability, scalability, and performance features in those account types.

A **General-purpose v1 (GPv1) account** provides access to all Azure Storage services, but may not have the latest features or the lowest per gigabyte pricing. For example, cool storage and archive storage are not supported in GPv1. However, hot and cool access tiers are supported in GPv2.

A **Blob storage account** is a specialized storage account for storing your unstructured data as blobs (objects) in Azure Storage. Blob storage has two tiers:

- A **Hot** access tier which indicates that the objects in the storage account will be more frequently accessed.
- A **Cool** access tier which indicates that the objects in the storage account will be less frequently accessed.

Standard storage are backed by magnetic devices. Best for applications where data is accessed infrequently. Premium storage are backed by solid state drives, best for high i/o like databases

## Replication Options

The data in your Microsoft Azure storage account [is always replicated](https://azure.microsoft.com/en-us/documentation/articles/storage-redundancy/) to ensure durability and high availability. When you create a storage account, you have four replication options.

| Replication Option                                   | Number of copies                     | Strategy                                                     |
| ---------------------------------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| Locally redundant storage (LRS)                      | Maintains three copies of your data. | Data is replicated three time within a single facility in a single region. |
| Zone-redundant storage (ZRS)                         | Maintains three copies of your data. | Data is replicated three times across two to three facilities, either within a single region or across two regions. |
| Geo-redundant storage (GRS)                          | Maintains six copies of your data.   | Data is replicated three times within the primary region, and is also replicated three times in a secondary region hundreds of miles away from the primary region. |
| Read access geo-redundant storage (RA-GRS) (Default) | Maintains six copies of your data.   | Data is replicated to a secondary geographic location, and also provides read access to your data in the secondary location. |

 By default, Azure storage accounts are configured to use **read-only redundant storage**. Read access geo-redundant storage replicates your data to a secondary geographic location, and also provides read access to your data in the secondary location.

You can change how your data is replicated after your storage account has been created, unless you specified ZRS when you created the account. However, you may incur an additional one-time data transfer cost if you switch from LRS to GRS or RA-GRS.

Blobs

Blob block storage is the most cost effective storage. Data is written in blocks and optimized for sequention I/O.

**CORS** : Azure storage supports CORS (cross origin resource sharing): You can configure storage to accept data coming from a different domain. You can do that in the browser now. That means you can write data directly to storage (instead of relying on an API that uploads byte stream of data into memory first.)

**Valet Key Pattern:** Often used with storage: refer https://www.youtube.com/watch?v=H0-_XCv7-A4

@timestamp: 10.09

![](..\assets\storage32.PNG)

![](..\assets\storage33.PNG)

![](..\assets\storage34.PNG)

# Virtual Machine Storage

![](..\assets\storage6.PNG)

![](..\assets\storage7.PNG)

![](..\assets\storage8.PNG)

![](..\assets\storage9.PNG)

![](..\assets\storage10.PNG)

![](..\assets\storage11.PNG)

![](..\assets\storage12.PNG)



Premium Storage is only supported on Azure DS, DSv2, GS, or FS series virtual machines

The maximum size for a file share is 5 TBs.

You can add virtual disks to every virtual machine to store application data or other data you need to keep. Data disks are registered as SCSI drives and are labeled with a letter that you choose.

![](..\assets\storage13.PNG)
Entity must contain a PartitionKey(customers, winephotos) and a RowKey (primary key for row). In JSON structure.
![](..\assets\storage14.PNG)
![](..\assets\storage15.PNG)
![](..\assets\storage16.PNG)
![](..\assets\storage17.PNG)
![](..\assets\storage18.PNG)
![](..\assets\storage19.PNG)
![](..\assets\storage20.PNG)
![](..\assets\storage21.PNG)
![](..\assets\storage22.PNG)
![](..\assets\storage23.PNG)
![](..\assets\storage24.PNG)
![](..\assets\storage26.PNG)
Azure backup uses cached files. You should allow at least 10% of the production server disk for caching. Azure will encrypt and decrypt your files using a passphrase that you supply. If you lose the passphrase your backup cannot be recovered.

To use Azure Backup you will need to download the Azure Backup Agent. When you install (register) the backup agent you will need the vault credentials.

![](..\assets\storage27.PNG)
![](..\assets\storage28.PNG)

![](..\assets\storage29.PNG)

![](..\assets\storage30.PNG)

![](..\assets\storage31.PNG)

By default, storage analytics retention is set to zero (0) days. This means storage metrics and logs are kept indefinitely and you are responsible for cleaning up the storage.