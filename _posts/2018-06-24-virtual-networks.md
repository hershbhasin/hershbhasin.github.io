---

layout: post
title: "Azure Virtual Networks"
author: "Hersh Bhasin"
comments: true
categories: paas AZ-300 Virtual-Networks
published: true

---

# Azure Virtual Networks

Study notes for [edx course: Azure Virtual Networks](https://courses.edx.org/courses/course-v1:Microsoft+AZURE203x+1T2019a/course/)

# Virtual networks

Azure Virtual Network is a fundamental component that acts as an organization’s network in Azure. Organizations can use virtual networks to connect resources. Virtual networks in Microsoft Azure are network overlays that you can use to configure and control connectivity between Azure resources such as VMs and load balancers.

![](..\assets\vnet1.PNG)

![](..\assets\vnet2.PNG)

### How many Virtual Networks?

By default, you can create up to 50 virtual networks per subscription per region, although you have the ability to increase this limit to 500 by contacting Azure support. These virtual networks are free of charge, but other dependent resources such as Public IP or application gateways are charged.

#### User Defined Routes

User Defined Routes (UDR) control network traffic by defining routes that specify the next hop of the traffic flow. You can assign User Defined Routes to virtual network subnets.

#### Forced tunneling

With forced tunneling you can redirect internet bound traffic back to the company’s on-premises infrastructure. Forced tunneling is commonly used in scenario where organizations want to implement packet inspection or corporate audit.

#### Regional virtual networks

Azure Virtual Network is bound to Azure subscriptions and it is not possible for multiple subscriptions to use the same Azure virtual network. If you need to provide communications between different Azure subscriptions, you need to create separate Azure virtual networks in each subscription and then use site-to-site virtual network connections or the Microsoft Azure service ExpressRoute, to connect them. All new virtual networks are regional virtual networks. This means that they can span a complete Azure region or datacenter. This differs from the legacy implementation of virtual networks in Azure, which were restricted to a single affinity group, allowing you to co-locate virtual networks, storage accounts, and services in the physical proximity to each other within the same area of a single datacenter. If you have older virtual networks in your subscription, these could be tied to an affinity group. However, over time, you need to migrate all virtual networks to regional virtual networks and remove their ties to specific affinity groups.

#### Cross-premises network connectivity

Virtual networks in Microsoft Azure also enable you to extend your on-premises networks to the cloud. To extend your on-premises network, you can create a virtual private network (VPN) between your on-premises computers or networks and an Azure virtual network. Alternatively, you can use ExpressRoute to provide a connection to an Azure virtual network that does not cross the Internet. Using these two methods, you can enable on-premises users to access Azure services as if they were physically located on-premises in your own datacenter.

To connect to an Azure virtual network from an on-premises network, you can use one of the following methods:

- **A point-to-site VPN**. This is a VPN that connects individual computers to an Azure virtual network. You must create a VPN connection from each on-premises computer that you want to connect to the Azure virtual network.
- **A site-to-site VPN**. This is a VPN that connects an on-premises network and all its computers to an Azure virtual network. To create this connection, you must configure a gateway and IP routing in the on-premises network; it is not necessary to configure individual on-premises computers.
- **ExpressRoute**. An ExpressRoute connection is a dedicated service that does not connect across the Internet. Instead, it uses a private connection to Azure datacenters, provided by a network provider. By using ExpressRoute, you can increase security, reliability, and bandwidth.
- You also can create a VPN that connects two Azure virtual networks. This is called a **VNet-to-VNet connection**.

Whenever you want to connect to an Azure virtual network, you must provision a VPN gateway in Azure. The VPN gateway routes traffic between VMs and PaaS cloud services in the virtual network, and computers at the other end of the connection.

For more information, you can see:
VPN Gateway: https://aka.ms/edx-azure203x-az7
ExpressRoute: https://aka.ms/edx-azure203x-az8



# IP addresses

VMs, Azure load balancers, and application gateways in a single virtual network require unique IP addresses in the same way as clients in an on-premises subnet do. This enables these resources to communicate with each other. There are two types of IP addresses that are used in a virtual network:

*Private IP addresses*. A private IP address is allocated to a VM dynamically or statically from the defined scope of IP addresses in the virtual network. This address is used by VMs in the virtual network to communicate with other VMs in the same virtual network connected VNets/networks through a gateway/ExpressRoute connection.

*Public IP addresses*. Public IP addresses allow Azure resources to communicate with external clients, and are assigned directly at the virtual network interface card of the VM or to the load balancer.

You can associate a public IP address resource with any of the following resources: 

- Virtual machines (VM)

- Internet-facing load balancers

- VPN gateways

- Application gateways

![](..\assets\vnet6.PNG)
![](..\assets\vnet9.PNG)
![](..\assets\vnet8.PNG)

Public IP can be assigned to VM, Load Balancer, Gateways(VPN & Application)

Public  IP cannot be Static for Gateways(VPN, Application)

Private IP can be assigned to VM, Load Balancer, Application gateway (both Static & Dynamic)

**Dynamic  IP** 

There are two methods in which an IP address is allocated to a public IP resource - dynamic or static. The default allocation method is dynamic, where an IP address is not allocated at the time of its creation. Instead, the public IP address is allocated when you start (or create) the associated resource (like a VM or load balancer). The IP address is released when you stop (or delete) the resource. This causes the IP address to change when you stop and start a resource.

Q: When is a dynamic IP address allocated?

A: Start or Create of resource

Explanation

The table below shows the specific property through which a public IP address can be associated to a top-level resource, and the possible allocation methods (dynamic or static) that can be used.

| Top-level resource  | IP address association   | Dynamic | Static |
| :------------------ | :----------------------- | :------ | :----- |
| Virtual machine     | Network interface        | Yes     | Yes    |
| Load balancer       | Front end configuration  | Yes     | Yes    |
| VPN gateway         | Gateway IP configuration | Yes     | No     |
| Application gateway | Front end configuration  | Yes     | No     |

**Static IP**

A Static IP cannot be assigned to a Application Gateway or a VPN Gateway

**IP Addressing in Virtual Networks**

When you create a virtual network, you define the scope of IP addresses that you can use for allocations to the networking resources. The scope of IP addresses can use both private IPv4 ranges and public IP ranges. You can use the following private IP address scopes:

The RFC 1918 standard defines three private address spaces that are never used for addressing on the Internet. Administrators use these ranges behind Network Address Translation (NAT) devices to ensure unique addresses used within intranets never prevent communication with Internet servers. These three address spaces were the only ones that are supported within an Azure VNet previously, but  now you can use public IP address ranges, and any IP address range defined in RFC 1918. See the following [update](https://azure.microsoft.com/en-us/updates/non-rfc-1918-space-is-now-allowed-in-a-virtual-network/).

The private address spaces are:      

- 10.0.0.0/8. This address space includes all addresses from 10.0.0.1 to 10.255.255.255.     
- 172.16.0.0/12. This address space includes all addresses from 172.16.0.1 to 172.31.255.255.      
- 192.168.0.0/16. This address space includes all addresses from 192.168.0.1 to 192.168.255.255.

When you specify an address space for a VNet, you usually specify a much smaller range within one of the private address spaces. For example, if you specified the address space 10.1.1.0/24, that means that all addresses from 10.1.1.1 to 10.1.1.255 should be routed into your VNet.

In a cloud-only virtual network, you can specify any address range from the RFC 1918 private spaces. However, if you connect to a VNet with a VPN or ExpressRoute, you must ensure that the address space is unique and does not overlap any of the ranges that are already in use on-premises or in other VNets.

There are a few IP address ranges that are not allowed:

- 224.0.0.0/4 (Multicast)
- 255.255.255.255/32 (Broadcast)
- 127.0.0.0/8 (loopback)
- 169.254.0.0/16 (link-local)
- 168.63.129.16/32 (Internal DNS)

# Subnets

You can further divide your network by using subnets for logical and security isolation of Azure resources. Each subnet contains a range of IP addresses that fall within the virtual network address space. VMs and PaaS role instances deployed to subnets (same or different) within a VNet can communicate with each other without any extra configuration. You can also configure route tables and NSGs to a subnet.

![](..\assets\vnet3.PNG)

**Subnet Allocation**

You must also sub-divide the VMs and cloud services in your VNet by providing one or more subnets. The range you specify for a subnet must be completely contained within its parent VNet’s address space. Within each subnet, the first three IP addresses and the last IP address are reserved and cannot be used for VMs or cloud services. The smallest subnets that are supported use a 29-bit subnet mask.

# Network interface card

VMs communicate with other VMs and other resources on the network by using virtual network interface cards (NICs). Virtual NICs configure VMs with private and optional public IP address. VMs can have more than one NIC for different network configurations.

### Multiple NICs in Virtual Machines

You can create virtual machines (VMs) in Azure and attach multiple network interfaces (NICs) to each of your VMs. Multi NIC is a requirement for many network virtual appliances, such as application delivery and WAN optimization solutions. Multi NIC also provides more network traffic management functionality, including isolation of traffic between a front end NIC and back end NIC(s), or separation of data plane traffic from management plane traffic.

![](..\assets\vnet10.PNG)

- Internet-facing VIP (classic deployments) is only supported on the "default" NIC. There is only one VIP to the IP of the default NIC.
- At this time, Instance Level Public IP (LPIP) addresses (classic deployments) are not supported for multi NIC VMs.
- The order of the NICs from inside the VM will be random, and could also change across Azure infrastructure updates. However, the IP addresses, and the corresponding ethernet MAC addresses will remain the same. For example, assume **Eth1** has IP address 10.1.0.100 and MAC address 00-0D-3A-B0-39-0D; after an Azure infrastructure update and reboot, it could be changed to **Eth2**, but the IP and MAC pairing will remain the same. When a restart is customer-initiated, the NIC order will remain the same.
- The address for each NIC on each VM must be located in a subnet and multiple NICs on a single VM can each be assigned addresses that are in the same subnet.
- The VM size determines the number of NICS that you can create for a VM.
- Multi NIC VMs must be created in Azure virtual networks (VNets). Non-VNet VMs cannot be configured with Multi NICs.

### Limitations of Multiple NICs

The following limitations are applicable when using the multi NIC feature:

- All VMs in an availability set need to use either multi NIC or single NIC. There cannot be a mixture of multi NIC VMs and single NIC VMs within an availability set. Same rules apply for VMs in a cloud service.
- A VM with single NIC cannot be configured with multi NICs (and vice-versa) once it is deployed, without deleting and re-creating it.

# Network security groups

You can use network security groups to provide network isolation for Azure resources by defining rules that can allow or deny specific traffic to individual VMs or subnets. This enables you to design your Azure virtual network to provide a network experience that is similar to an on-premises network. You can achieve the same functionality in your Azure virtual network as you would in the on-premises networks, such as perimeter ed zone).

### Network Security Group Rules

Put a least restrictive rule with a higher priority, so it gets evaluated first, and a more restrictive rule with a lower priority, so that it gets evaluated further down a chain.

![](..\assets\vnet4.PNG)

![](..\assets\vnet5.PNG)

NSGs contain rules that specify whether the traffic is approved or denied. Each rule is based on a source IP address, a source port, a destination IP address, and a destination port. Based on whether the traffic matches this combination, it either is allowed or denied. Each rule consists of the following properties:

- **Name**. This is a unique identifier for the rule.
- **Direction**. Direction specifies whether the traffic is inbound or outbound.
- **Priority**. If multiple rules match the traffic, rules with higher priority apply.
- **Access**. Access specifies whether the traffic is allowed or denied.
- **Source IP address prefix**. This identifies from where traffic originates. This prefix can be based on a single IP address, a range of IP addresses in CIDR notation, or the asterisk (*) wildcard character, that must match all possible IP addresses.
-  **Source port range**. This specifies source ports by using either a single port number from 1-65535, a range of ports (200-400), or the asterisk (*) wildcard character that denotes all possible ports.
- **Destination IP address prefix**. This identifies the traffic destination based on a single IP address, a range of IP addresses in CIDR notation, or the asterisk (*) wildcard character, that must match all possible IP addresses.
- **Destination port range**. This specifies destination ports by using either a single port number from 1-65535, a range of ports (200-400), or the asterisk (*) wildcard character, that denotes all possible ports.
- **Protocol**. Protocol specifies a protocol that matches the rule. It can be UDP, TCP, or the asterisk (*) wildcard character *.

There are **predefined default rules** for inbound and outbound traffic. You cannot delete these rules, but you can override them, because they have the lowest priority.

 Default rules:

* allow all inbound and outbound traffic within a virtual network, 
* allow outbound traffic towards the Internet, 
* and permit inbound traffic to Azure load balancer. 
* There is also a default rule in both inbound and outbound sets of rules that denies all network communication with the lowest priority.

### Custom Network Security Group Rules

There are predefined default rules for inbound and outbound traffic. You cannot delete these rules, but you can override them, because they have the lowest priority. Default rules allow all inbound and outbound traffic within a virtual network, allow outbound traffic towards the Internet, and permit inbound traffic to Azure load balancer. There is also a default rule in both inbound and outbound sets of rules that denies all network communication with the lowest priority.

When you create a custom rule, you can use default tags in the source and destination address prefix to specify a predefined category of IP addresses. These default tags are:

- **Internet**. This tag represents Internet IP addresses.
- **Virtual_network**. This tag identifies all IP addresses that are defined in the IP range for the virtual network. It also includes IP address ranges from on-premises networks when they are defined as Local network to virtual network.
- **Azure_loadbalancer**. This tag specifies the default Azure load balancer destination.



### Planning Network Security Groups

You can design Network Security Groups to isolate virtual networks in security zones, similar to the model that is used in on-premises infrastructure. You can apply network security groups to subnets so that you can create protected screened subnets (also called DMZ) that can restrict traffic flow to all the machines that reside within that subnet. You also can assign network security groups to individual computers in the Azure classic deployment model to control traffic that is both destined for and leaving the VM. In an Azure Resource Manager deployment, you can assign network security groups to a NIC so that only the traffic that flows through that NIC is controlled by network security group rules. If the VM has multiple NICs, network security group rules are not automatically applied to traffic that is designated to other NICs.

Network security groups are resources that are created in a resource group, but can be shared with other resource groups that exist in your subscription. This means that if you create a network security group, for example in the TestRG resource group, you can use that network security group for a VM that belong to another resource group, for example ProductionRG.

Some important things to keep in mind while implementing network security groups include:

- By default you can create 5000 NSGs per region per subscription.
- You can apply only one NSG to a VM, subnet, or NIC.
- By default, you can have up to 1000 rules in a single NSG. You can raise this limit to 500 by contacting Azure support.
- You can apply an NSG to multiple resources.

Subnet, then NIC is evaluated for NSG


# Name Resolution (DNS)

Refer: https://docs.microsoft.com/en-us/azure/dns/dns-delegate-domain-azure-dns

DNS domains in Azure DNS are hosted on Azure’s global network of DNS name servers. Azure DNS uses **Anycast networking** so that each DNS query is answered by the closest available DNS server. This provides both fast performance and high availability for your domain.

Name resolution is the process by which a computer name is resolved to an IP address. A computer can use the IP address to connect to the named computer by using the IP address, which the user may find difficult to remember.

The Domain Name System (DNS) enables clients to resolve user-friendly fully qualified domain names (FQDNs), such as www.adatum.com, to IP addresses. Azure provides a DNS system to support many name resolution scenarios. However, in some cases, such as hybrid connection you might need to configure an external DNS system to provide name resolution for virtual machines on a virtual network.

Names of resources that are created in Azure can be resolved by using Azure-provided name resolution or by using a customer provided DNS server. For example, a VM can use the Azure-provided DNS to resolve the name of any other VM in the same virtual network. However, in a hybrid scenario where your on-premises network is connected to an Azure virtual network through a VPN or ExpressRoute circuit, an on-premises computer cannot resolve the name of a VM in an Azure virtual network until you configure the DNS servers with a record for the VM. Furthermore, resources created in the same virtual network and deployed with Azure Resource Manager (ARM) share the same DNS suffix; therefore, in most cases name resolution by using FQDN is not required. For virtual networks that are deployed by using the Azure classic deployment model, the DNS suffix is shared among VMs that belong to the same cloud service. Therefore, name resolution between VMs that belong to different cloud services in the same virtual network require the use of FQDN.

![](..\assets\vnet7.PNG)

If you are planning to use your own DNS system, you must ensure that all computers can reach a DNS server for registering and resolving IP addresses. You can either deploy DNS on a VM in the Azure VNet or have VM register their addresses with an on-premises DNS server. Your DNS server must meet the following requirements:

- The server must support Dynamic DNS (DDNS) registration.
- The server must have record scavenging switched off. Because DHCP leases in an Azure VNet are infinite, record scavenging can remove records that have not been renewed but are still correct.
- The server must have DNS recursion enabled.
- The server must be accessible on TCP/UDP port 53 from all clients.

Note: *If two VMs are deployed in different IaaS cloud services but not in a VNet, they cannot communicate at all, even by using DIPs. Therefore name resolution is not applicable.*

![](..\assets\vnet11.PNG)

![](..\assets\vnet12.PNG)



In above example, www.contoso.com is the DNS Zone, The two IP addresses are the Record Set



DNS Zones:  Examples: sales.contoso.com, production.contoso.com  (Subdomains?)

Record Sets: Example DNS Round Robin; poor man's load balancer; create copies of same web app that run on different IPs, create entries for the app and have the DNS server hand out the app, alternating in a Round Robin.

![](..\assets\vnet13.PNG)

![](..\assets\vnet14.PNG)

### Resolution and Delegation

There are two types of DNS servers:

- An *authoritative* DNS server hosts DNS zones. It answers DNS queries for records in those zones only.

- A *recursive* DNS server does not host DNS zones. It answers all DNS queries by calling authoritative DNS servers to gather the data it needs.

  > Azure DNS provides an authoritative DNS service. It does not provide a recursive DNS service. Cloud Services and VMs in Azure are automatically configured to use a recursive DNS that is provided separately as part of Azure's infrastructure.

DNS clients in PCs or mobile devices typically call a recursive DNS server to perform any DNS queries the client applications need.

When a recursive DNS server receives a query for a DNS record such as ‘www.contoso.com’, it first needs to find the name server hosting the zone for the ‘contoso.com’ domain. To do this, it starts at the root name servers, and from there finds the name servers hosting the ‘com’ zone. It then queries the ‘com’ name servers to find the name servers hosting the ‘contoso.com’ zone. Finally, it is able to query these name servers for ‘www.contoso.com’.

This is called resolving the DNS name. Strictly speaking, DNS resolution includes additional steps such as following CNAMEs, but that’s not important to understanding how DNS delegation works.

How does a parent zone ‘point’ to the name servers for a child zone? It does this using a special type of DNS record called an NS record (NS stands for ‘name server’). For example, the root zone contains NS records for 'com' and shows the name servers for the ‘com’ zone. In turn, the ‘com’ zone contains NS records for ‘contoso.com’, which shows the name servers for the ‘contoso.com’ zone. Setting up the NS records for a child zone in a parent zone is called delegating the domain.

![](..\assets\vnet25.PNG)

Each delegation actually has two copies of the NS records; one in the parent zone pointing to the child, and another in the child zone itself. The ‘contoso.com’ zone contains the NS records for ‘contoso.com’ (in addition to the NS records in ‘com’). These are called authoritative NS records and they sit at the apex of the child zone.

### Delegating a Domain to Azure DNS

Once you create your DNS zone in Azure DNS, you need to set up NS records in the parent zone to make Azure DNS the authoritative source for name resolution for your zone. For domains purchased from a registrar, your registrar will offer the option to set up these NS records.

For example, suppose you purchase the domain ‘contoso.com’ and create a zone with the name ‘contoso.com’ in Azure DNS. As the owner of the domain, your registrar will offer you the option to configure the name server addresses (that is, the NS records) for your domain. The registrar will store these NS records in the parent domain, in this case ‘.com’. Clients around the world will then be directed to your domain in Azure DNS zone when trying to resolve DNS records in ‘contoso.com’.

For example, suppose you purchase the domain ‘contoso.com’ and create a zone with the name ‘contoso.com’ in Azure DNS. As the owner of the domain, your registrar will offer you the option to configure the name server addresses (that is, the NS records) for your domain. The registrar will store these NS records in the parent domain, in this case ‘.com’. Clients around the world will then be directed to your domain in Azure DNS zone when trying to resolve DNS records in ‘contoso.com’.

### Finding the Name Server Names

Before you can delegate your DNS zone to Azure DNS, you first need to know the name server names for your zone. Azure DNS allocates name servers from a pool each time a zone is created.

The easiest way to see the name servers assigned to your zone is via the Azure porta (Properties)l. In this example, the zone ‘contoso.net’ has been assigned name servers ‘ns1-01.azure-dns.com’, ‘ns2-01.azure-dns.net’, ‘ns3-01.azure-dns.org’, and ‘ns4-01.azure-dns.info’:

Azure DNS automatically creates authoritative NS records in your zone containing the assigned name servers. To see the name server names via Azure PowerShell or Azure CLI, you simply need to retrieve these records.

```powershell
 $zone = Get-AzureRmDnsZone –Name contoso.net –ResourceGroupName MyResourceGroup
 Get-AzureRmDnsRecordSet –Name “@” –RecordType NS –Zone $zone
```

### Setting Up Delegation

Each registrar has their own DNS management tools to change the name server records for a domain. In the registrar’s DNS management page, edit the NS records and replace the NS records with the ones Azure DNS created.

When delegating a domain to Azure DNS, you must use the name server names provided by Azure DNS. You should always use all 4 name server names, regardless of the name of your domain. Domain delegation does not require the name server name to use the same top-level domain as your domain.

You should not use 'glue records' to point to the Azure DNS name server IP addresses, since these IP addresses may change in future. Delegations using name server names in your own zone, sometimes called 'vanity name servers', are not currently supported in Azure DNS.

### Delegating Sub-Domains in Azure DNS

If you want to set up a separate child zone, you can delegate a sub-domain in Azure DNS. For example, having set up and delegated ‘contoso.com’ in Azure DNS, suppose you would like to set up a separate child zone, ‘partners.contoso.com’.

Setting up a sub-domain follows a similar process as a normal delegation. The only difference is that in step 3 the NS records must be created in the parent zone ‘contoso.com’ in Azure DNS, rather than being set up via a domain registrar.

1. Create the child zone ‘partners.contoso.com’ in Azure DNS.
2. Look up the authoritative NS records in the child zone to obtain the name servers hosting the child zone in Azure DNS.
3. Delegate the child zone by configuring NS records in the parent zone pointing to the child zone.

### To delegate a sub-domain

The following PowerShell example demonstrates how this works. The same steps can be executed via the Azure Portal, or via the cross-platform Azure CLI.

#### Step 1. Create the parent and child zones

First, we create the parent and child zones. These can be in same resource group or different resource groups.

```
$parent = New-AzureRmDnsZone -Name contoso.com -ResourceGroupName RG1
$child = New-AzureRmDnsZone -Name partners.contoso.com -ResourceGroupName RG1
```

#### Step 2. Retrieve NS records

Next, we retrieve the authoritative NS records from child zone as shown in the next example. This contains the name servers assigned to the child zone.

```
$child_ns_recordset = Get-AzureRmDnsRecordSet -Zone $child -Name "@" -RecordType NS
```

#### Step 3. Delegate the child zone

Create corresponding NS record set in the parent zone to complete the delegation. Note that the record set name in the parent zone matches the child zone name, in this case "partners".

```
$parent_ns_recordset = New-AzureRmDnsRecordSet -Zone $parent -Name "partners" -RecordType NS -Ttl 3600
$parent_ns_recordset.Records = $child_ns_recordset.Records
Set-AzureRmDnsRecordSet -RecordSet $parent_ns_recordset
```

### To Verify Name Resolution is Working

After completing the delegation, you can verify that name resolution is working by using a tool such as ‘nslookup’ to query the SOA record for your zone (which is also automatically created when the zone is created).

Note that you do not have to specify the Azure DNS name servers, since the normal DNS resolution process will find the name servers automatically if the delegation has been set up correctly.

```
nslookup –type=SOA contoso.com

Server: ns1-04.azure-dns.com
Address: 208.76.47.4

contoso.com
primary name server = ns1-04.azure-dns.com
responsible mail addr = msnhst.microsoft.com
serial = 1
refresh = 900 (15 mins)
retry = 300 (5 mins)
expire = 604800 (7 days)
default TTL = 300 (5 mins)
```

### DNS Record Types

An Azure DNS zone can support all common DNS record types, such as A, AAAA, CNAME, MX, NS, SOA, SRV and TXT. The following table describes the function of each type of record.

![](..\assets\vnet26.PNG)
# Distributing Network Traffic

There are different options to distribute network traffic using Microsoft Azure. These options work differently from each other, have different feature sets,  and support different scenarios. They can each be used in isolation or in combination. These are:

Note: Reverse Proxy/Reverse Nat/ or port translation means: Example listen on port 80 but internally go to port 8888

![](..\assets\vnet16.PNG)

![](..\assets\vnet17.PNG)

![](..\assets\vnet18.PNG)

## Azure Load Balancer

(Typically used for VM Scale Sets)

To increase availability and scalability, you can create two or more VMs that publish the same application. For example, if three VMs host the same website, you might want to distribute incoming traffic between them and ensure that if one VM fails, traffic is distributed automatically to the other two. You can use an Azure load balancer to enable this traffic distribution between VMs. In this configuration, a single endpoint is shared between multiple VMs or services. For example, you can spread the load of web request traffic across multiple web servers.

You can use two types of Azure load balancers:

*Public load balancer*. A public load balancer maps the public IP address and port number of incoming traffic to the private IP address and port number of the virtual machine and vice versa for the response traffic from the virtual machine.

*Internal load balancer*. Internal Load Balancer only directs traffic to resources that are inside a virtual network or that use a VPN to access Azure infrastructure. In this respect, internal Load Balancer differs from a public Load Balancer. Azure infrastructure restricts access to the load-balanced frontend IP addresses of a virtual network. Frontend IP addresses and virtual networks are never directly exposed to an internet endpoint. Internal line-of-business applications run in Azure and are accessed from within Azure or from on-premises resources.

![](..\assets\vnet19.PNG)

![](..\assets\vnet20.PNG)

### Hash-Based Distribution

The load balancer uses a hash-based distribution algorithm. By default, it uses a 5-tuple (source IP, source port, destination IP, destination port, and protocol type) hash to map traffic to available servers. It provides stickiness only *within* a transport session. Packets in the same TCP or UDP session will be directed to the same datacenter IP instance behind the load-balanced endpoint. When the client closes and reopens the connection or starts a new session from the same source IP, the source port changes. This may cause the traffic to go to a different datacenter IP endpoint.

![](..\assets\vnet27.PNG)

### Port Forwarding

The load balancer gives you control over how inbound communication is managed. This communication can include traffic that's initiated from Internet hosts or virtual machines in other cloud services or virtual networks. This control is represented by an endpoint (also called an input endpoint).

An endpoint listens on a public port and forwards traffic to an internal port. You can map the same ports for an internal or external endpoint or use a different port for them. For example, you can have a web server configured to listen to port 81 while the public endpoint mapping is port 80. The creation of a public endpoint triggers the creation of a load balancer instance.

The default use and configuration of endpoints on a virtual machine that you create by using the Azure portal are for the Remote Desktop Protocol (RDP) and remote Windows PowerShell session traffic. You can use these endpoints to remotely administer the virtual machine over the Internet.

### Service Monitoring

The load balancer can probe the health of the various server instances. When a probe fails to respond, the load balancer stops sending new connections to the unhealthy instances. Existing connections are not impacted.

Three types of probes are supported:

- Guest agent probe (on PaaS VMs only)
- HTTP custom probe
- TCP custom probe



## Traffic Manager

(Load Balancer for Azure Regions)

**The most important point to understand is that Traffic Manager works at the DNS level.** Traffic Manager uses DNS to direct end users to particular service endpoints, based on the chosen traffic-routing method and the current endpoint health. Clients then connect to the selected endpoint **directly**. Traffic Manager is not a proxy, and does not see the traffic passing between the client and the service.



Microsoft Azure Traffic Manager is another load-balancing solution that is included within Azure. You can use Traffic Manager to load balance between endpoints that are located in :

1. different Azure regions, 
2. at hosted providers, 
3. or in on-premises datacenters.

These endpoints can include Azure VMs and Azure websites. You can configure this load-balancing service to support priority or to ensure that users connect to an endpoint that is close to their physical location for faster response.

Microsoft Azure Traffic Manager allows you to control the distribution of user traffic to your service endpoints running in different datacenters around the world.

Service endpoints supported by Traffic Manager include Azure VMs, Web Apps, and cloud services. You can also use Traffic Manager with external, non-Azure endpoints.

Traffic Manager works by using the Domain Name System (DNS) to direct end-user requests to the most appropriate endpoint, based on the configured traffic-routing method and current view of endpoint health. Clients then connect to the appropriate service endpoint directly.

Traffic Manager supports a range of traffic-routing methods to suit different application needs. Traffic Manager provides endpoint health checks and automatic endpoint failover, enabling you to build high-availability applications that are resilient to failure, including the failure of an entire Azure region.

![](..\assets\vnet21.PNG)

![](..\assets\vnet22.PNG)



### Traffic Manager Example

Contoso Corp have developed a new partner portal. The URL for this portal will be https://partners.contoso.com/login.aspx. The application is hosted in Azure, and to improve availability and maximize global performance, they wish to deploy the application to 3 regions worldwide and use Traffic Manager to distribute end users to their closest available endpoint.

To achieve this configuration:

- They deploy 3 instances of their service. The DNS names of these deployments are ‘contoso-us.cloudapp.net’, ’contoso-eu.cloudapp.net’, and ‘contoso-asia.cloudapp.net’.

- They then create a Traffic Manager profile, named ‘contoso.trafficmanager.net’, which is configured to use the ‘Performance’ traffic-routing method across the 3 endpoints named above.

- Finally, they configure their vanity domain, ‘partners.contoso.com’ to point to ‘contoso.trafficmanager.net’, using a DNS CNAME record.

  

![](..\assets\vnet28.PNG)

## Application Gateway

One of the common uses for Application Gateway is in the situation when I have multiple web servers configured for http and I don't want to put certificates on all of them. I would then have a Application Gateway listening on the SSL port and transfer the request over http to underlying web servers; when response comes from underlying web servers, the process is reversed. 

Application gateways provide load-balanced solutions for network traffic that is based on the HTTP protocol. They use routing rules as application-level policies that can offload Secure Sockets Layer (SSL) processing from load-balanced VMs. In addition, you can use application gateways for a cookie-based session affinity scenario.

Microsoft Azure Application Gateway provides an Azure-managed HTTP load-balancing solution based on layer-7 load balancing.

Application load balancing enables IT administrators and developers to create routing rules for network traffic based on HTTP. The Application Gateway service is highly available and metered.

Application Gateway supports layer-7 application delivery for the following:

- HTTP load balancing
- Cookie-based session affinity
- [Secure Sockets Layer (SSL) offload](https://azure.microsoft.com/en-us/documentation/articles/application-gateway-ssl-arm/)
- [URL-based content routing](https://azure.microsoft.com/en-us/documentation/articles/application-gateway-url-route-overview/)
- [Multi-site routing](https://azure.microsoft.com/en-us/documentation/articles/application-gateway-multi-site-overview/)

![](..\assets\vnet23.PNG)

![](..\assets\vnet24.PNG)

# Intersite Connectivity

![](..\assets\vnet29.PNG)

![](..\assets\vnet30.PNG)

![](..\assets\vnet31.PNG)

## A Gateway Subnet is added to a subnet and a VPN Gateway is deployed to a Gateway Subnet

![](..\assets\vnet32.PNG)

![](..\assets\vnet33.PNG)

![](..\assets\vnet34.PNG)

![vnet36](..\assets\vnet36.PNG)

![vnet36](..\assets\vnet37.PNG)

![vnet36](..\assets\vnet38.PNG)