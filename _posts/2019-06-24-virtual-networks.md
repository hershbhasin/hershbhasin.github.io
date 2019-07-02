---

layout: post
title: "AZ 300 Azure Virtual Networks Study Notes"
author: "Hersh Bhasin"
comments: true
categories: paas AZ-300 Virtual-Networks
published: true

---

# AZ 300 Azure Virtual Networks Study Notes

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

## Traffic Manager

(Load Balancer for Azure Regions)

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

# Name Resolution (DNS)

Refer: https://docs.microsoft.com/en-us/azure/dns/dns-delegate-domain-azure-dns

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