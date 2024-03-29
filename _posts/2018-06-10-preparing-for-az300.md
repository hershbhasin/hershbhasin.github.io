---
layout: post
title: Azure AZ 300 Test Preparation
author: Hersh Bhasin
category: AZ-300
permalink: /azure-az300-prep/
---

# Proposed Strategy for preparing for AZ300

The strategy I am thinking of using for test prep is to use free [eDX Azure Resources](https://www.edx.org/learn/azure) as far as possible , and supplement with MS documentation where it falls short.  My proposed sequence of  study is as follows , having reorganized the [AZ 300 Syllabus (Microsoft)](https://www.microsoft.com/en-us/learning/exam-az-300.aspx) in a manner that makes sense to me :

I will be checking off the items that are covered by edX training.

# Section: Deploy and configure infrastructure (25-30%)

### Virtual Machines

[Edx: Virtual Machines](https://www.edx.org/course/microsoft-azure-virtual-machines-3)

[Hersh's Study Notes](https://hershbhasin.com/2018-06-15/az300-vm)

**Create and configure a Virtual Machine (VM) for Windows and Linux**

- [x] Create and configure a Virtual Machine (VM) for Windows and Linux
- [x] configure high availability
- [x] configure monitoring, networking, storage, and virtual machine size
- [x] deploy and configure scale sets

**Automate deployment of Virtual Machines (VMs)**

- [x] Modify Azure Resource Manager template
- [x] configure location of new VMs
- [x] configure VHD template
- [x] deploy from template
- [x] save a deployment as an Azure Resource Manager template
- [x] deploy Windows and Linux VMs

**Implement solutions that use virtual machines (VM)**

[edx: Automating Azure Workloads](https://www.edx.org/course/automating-azure-workloads-3)

- [x] provision VMs
- [x] create Azure Resource Manager templates
- [x] configure Azure Disk Encryption for VMs

### Microsoft Azure Virtual Networks

[edX Microsoft Azure Virtual Networks](https://www.edx.org/course/microsoft-azure-virtual-networks-3)

[Hersh's Notes: Microsoft Azure Virtual Networks](https://hershbhasin.com/2018-06-24/virtual-networks)



**Create connectivity between virtual networks**

- [x] create and configure VNET peering
- [x] create and configure VNET to VNET
- [x] verify virtual network connectivity
- [x] create virtual network gateway

**Implement and manage virtual networking**

- [x] configure private and public IP addresses, network routes, network interface, subnets, and virtual network

### Microsoft Azure Identity

[edx: Microsoft Azure Identity](https://www.edx.org/course/microsoft-azure-identity-3)

[ Plualsight: Savill](https://app.pluralsight.com/library/courses/microsoft-azure-managing-active-directory/table-of-contents)

[Study Notes](https://hershbhasin.com/2018-07-03/active-directory)

**Manage Azure Active Directory (AD)**

- [x] add custom domains 
- [x] configure Azure AD Identity Protection, Azure AD Join, and Enterprise State Roaming
- [x] configure self-service password reset
- [x] implement conditional access policies
- [x] manage multiple directories
  perform an access review

**Implement and manage hybrid identities**

- [x] install and configure Azure AD Connect
- [ ] configure federation and single sign-on
- [x] manage Azure AD Connect
- [x] manage password sync and writeback

### Microsoft Azure Storage

[edX: Microsoft Azure Storage](https://www.edx.org/course/microsoft-azure-storage-5)

[Study Notes](https://hershbhasin.com/2018-08-03/azure-storage)

**Create and configure storage accounts**

- [x] configure network access to the storage account
- [x] create and configure storage account
- [x] generate shared access signature
- [x] install and use Azure Storage Explorer
- [x] manage access keys
- [x] monitor activity log by using Azure Monitor logs
- [x] implement Azure storage replication

### Analyze resource utilization and consumption

No edx course available for this.

[Pluralsight: Monitoring Microsoft Azure Resources and Workloads](https://app.pluralsight.com/library/courses/microsoft-azure-resources-workloads-monitoring/table-of-contents)

- [x] configure diagnostic settings on resources
- [x] create baseline for resources
- [x] create and rest alerts
- [x] analyze alerts across subscription
- [x] analyze metrics across subscription
- [ ] create action groups
- [ ] monitor for unused resources
- [ ] monitor spend
- [ ] report on spend
- [x] utilize Log Search query functions
- [x] view alerts in Azure Monitor logs

#  Implement workloads and security (20-25%)

**Migrate servers to Azure**

[edx: Migrating Workloads to Azure](https://www.edx.org/course/automating-azure-workloads-3)

- [ ] migrate by using Azure Site Recovery
- [ ] migrate using P2V
- [x] configure storage
- [x] create a backup vault
- [ ] prepare source and target environments
- [x] backup and restore data
- [ ] deploy Azure Site Recovery agent
- [x] prepare virtual network

**Configure serverless computing**

[Pluralsight: Configuring Serverless Computing in Microsoft Azure](https://app.pluralsight.com/library/courses/microsoft-azure-serverless-computing-configuring/table-of-contents)

- [ ] manage a Logic App resource
- [ ] manage Azure Function app settings
- [ ] manage Event Grid
- [ ] manage Service Bus

**Implement application load balancing**

- [ ] configure application gateway and load balancing rules
- [ ] implement front end IP configurations
- [ ] manage application load balancing

**Integrate on-premises network with Azure virtual network**

- [ ] create and configure Azure VPN Gateway
- [ ] create and configure site to site VPN
- [ ] configure Express Route
- [ ] verify on-premises connectivity
- [ ] manage on-premises connectivity with Az

# Create and deploy apps (5-10%)

**Create web apps by using PaaS**

[edx: Microsoft Azure App Service](https://www.edx.org/course/microsoft-azure-app-service-3)

- [ ] create an Azure App Service Web App
- [ ] create documentation for the API
- [ ] create an App Service Web App for containers
- [ ] create an App Service background task by using WebJobs
- [ ] enable diagnostics logging

**Design and develop apps that run in containers**

[Pluralsight course: Azure Kubernetes Service (AKS) - The Big Picture](https://app.pluralsight.com/library/courses/azure-container-service-big-picture)

[Hersh's Study Notes: Azure Kubernetes Service (AKS) - The Big Picture](https://hershbhasin.com/2018-06-27/aks)

- [x] configure diagnostic settings on resources
- [x] create a container image by using a Docker file
- [x] create an Azure Kubernetes Service
- [x] publish an image to the Azure Container Registry
- [x] implement an application that runs on an Azure Container Instance
- [x] manage container settings by using code

# Implement authentication and secure data (5-10%)

[edx: Azure Security & Compliance](https://www.edx.org/course/azure-security-and-compliance-3)

**Implement authentication**

- [ ] implement authentication by using certificates, forms-based authentication, tokens, or Windows-integrated authentication
- [ ] implement multi-factor authentication by using Azure AD
- [ ] implement OAuth2 authentication
- [ ] implement Managed identities for Azure resources Service Principal authentication

**Implement secure data solutions**

- [ ] encrypt and decrypt data at rest and in transit
- [ ] encrypt data with Always Encrypted
- [ ] implement Azure Confidential Compute and SSL/TLS communications
- [ ] create, read, update, and delete keys, secrets, and certificates by using the KeyVault API

# Develop for the cloud and for Azure storage (20-25%)

### Cosmos Db

[edx: Developing Planet-Scale Applications in Azure Cosmos DB](https://www.edx.org/course/developing-planet-scale-applications-in-azure-cosmos-db-2)

**Develop solutions that use Cosmos DB storage**

- [ ] create, read, update, and delete data by using appropriate APIs
- [ ] implement partitioning schemes
- [ ] set the appropriate consistency level for operations

### SQL Server

[edx: Databases in Azure](https://www.edx.org/course/databases-in-azure-3)

**Develop solutions that use a relational database**

- [ ] provision and configure relational databases
- [ ] configure elastic pools for Azure SQL Database
- [ ] create, read, update, and delete data tables by using code

### Data Streams

[edx: processing Real-Time Data Streams in Azure](https://www.edx.org/course/processing-real-time-data-streams-in-azure-2)

**Configure a message-based integration architecture**

- [ ] configure an app or service to send emails, Event Grid, and the Azure Relay Service
- [ ] create and configure Notification Hub, Event Hub, and Service Bus
- [ ] configure queries across multiple products

**Develop for autoscaling**

- [ ] implement autoscaling rules and patterns (schedule, operational/system metrics, code that addresses singleton application instances)
- [ ] implement code that addresses transient state

## Links for preparing for AZ 300 Azure Certification

[AZ 300 Syllabus (Microsoft)](https://www.microsoft.com/en-us/learning/exam-az-300.aspx)

[Pixal Robots : Study resources for the AZ-300](https://pixelrobots.co.uk/2018/09/study-resources-for-the-az-300/)

[Udemy Practice Questions](https://www.udemy.com/az-300-azure-architecture-technologies-practice-test)