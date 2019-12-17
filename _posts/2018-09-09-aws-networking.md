---
layout: post
title: "AWS Networking"
author: "Hersh Bhasin"
comments: true
categories: AWS
permalink: /aws-networking/
---



Study notes on [Pluralsight AWSNetworking]( https://app.pluralsight.com/library/courses/aws-networking-deep-dive-vpc/table-of-contents )

![](..\assets\aws-vpc-1.PNG)

# web-vpc (VPC #1 Public VPC)

![](..\assets\aws-vpc-0.PNG)

![](..\assets\aws-vpc-0b.PNG)

**VPC**

Name: web-vpc

CIDR: 10.1.0.0/16

**Public subnet**

Name: web-pub

CIDR: 10.1.254.0/24

Zone: us-east-1a

**Internet Gateway**

Go to the internet gateways tab and create

Name: web-igw

Attach to vpc: web-vpc



**Route Table** (aka "Implied Router")

Each VPC has a default route table which is automatically associated with each subnet you create. All traffic into, out of and within a vpc traverses a implied router. It is the gatekeeper and the brains of vpc networking

*Create a route table and associate it with a vpc*

​	Name: web-pub, 

​	select the vpc to associate it with

*Associate subnet to route table*

​	Now in vpc, go to subnet associations tab and associate subnet 

​	Associated subnet: web-pub (10.1.254.0/24)

What this has done is that the aws has now assigned the "implied router" an ip address of 10.1.254.1 (it always gets the first address in the subnet associated with the route table: in this case web-pub subnet)

So each vpc has a "pingable" implicit router.

Analogy to a traditional network: this is like when your internet service provider places an internet router at your site: they configure it for their site, and all you have to do is connect your network to it, which is what we are going to do now. We will connect our internet gateway to the implicit router by adding a route for our internet gateway.

1. Go to route tables
2. Select web-pub route
3. Add route: Destination = 0.0.0.0/0 Target: select internet gateway: web-igw

Idea here is that if our web server tries to send a packet to an ip address out on the internet, that destination will match the default route (0.0.0.0/0 which means all destinations), and because the route table has the internet as the target, it will forward the packet via the internet gateway

Hence the analogy: creating a default route, pointing to the internet gateway is kind of like connecting to an ISP's internet router to your network.  

##  Launching an Instance



**Security Group**

Group name: web-pub-sg

Rules:

Protocol/port		 Source

ssh (TCP/22)			Your IP

HTTP (TCP/80)		0.0.0.0/0 (all ips)



**Elastic Network Interface (ENI)**  (network card)

Name: www1.eth0

Subnet: web-pub

IP Address: 10.1.244.10

Security group: web-pub-sg



**Elastic IP (EIP)**

Allocate new address (click)

Associate  address (select)

Resource type: Network interface

Network interface: www1.eth0

Private ip: select only ip address from drop down

**Launch Instance**

Instances/Launch instance

Community AMIs: search for ami-1cb6b467 (this include  docker )

Instance Type: t2.micro

Configure Details Tab:

1. Network: web-vpc

2. subnet: web-pub

3. (ENI) Network Interfaces: eth0
   1. Network Interface:  www1.eth0



```bash
ssh -i "pizza-keys.pem" ec2-user@34.206.56.131
```

notice that the implicit router is first ip of the subnet: 10.1.254.10

```bash
#ping implicit router
ping 10.1.254.10
#ping google to verify internet connectivity
ping 8.8.8.8

#to see routes (not the vpc route table but the routes of the instance)
route
```

![](..\assets\aws-vpc-2.PNG)

Note default route is 10.1.254.1 . This is the implied router. Whenever instance wants to send traffic  (say ping 8.8.8.8 --google), it checks the ip routing  table to see if route exists, if not it will use the default route, which is the implied router. when the implied router receives the request, it will check the vpc route table. Now the vpc route table has the rule: Destination = 0.0.0.0/0  (all ips) ;Target: internet gateway: web-igw. So the traffic from instance is directed to internet gateway.

```bash
ping 8.8.8.8 -> use implicit router: 10.1.254.1 -> use  internet gateway: web-igw.
```



Then NAT happens (see below)

implicit routers ip -> becomes Public Elastic IP

```bash
i.e. 10.1.254.1 ->  34.206.56.131
```



*NAT Translation*

AWS does network address translation (NAT). When you associate a private IP address to a Public IP Address, that enables network translation between them. The NAT translation seems to appear between the implied router and the internet gateway

```bash
# get your instances public ip
curl ifconfig.co
#result 34.206.56.131 which is the ip of your public Elastic IP

# but if we do a ip a, notice that the public ip address is not configured. Hence AWS is doing network adress translation NAT
ip a

#if you do ifconfig the eth0 inet addr is the implicit router address : 10.1.254.1S
```

![](..\assets\aws-vpc-3.PNG)



# Private VPC (shared-vpc for database)

![](..\assets\aws-vpc-4.PNG)

![](..\assets\aws-vpc-0c.PNG)

## VPC

Name: share-vpc

ipv4 cidr: 10.2.0.0/16

**Subnet**

Name: database

CIDR: 10.2.2.0/24

VPC: share-vpc

**Route Table**

Name: shared

VPC: shared-vpc

**Subnet Association**

Associate database subnet with route table "shared"

**Database** Instance

EC2/Launch Instance / Community AMIs/search for: ami-1cb6b467 (same image as before, i.e. image with docker)

* Network: shared-vpc

* subnet: database

* Network interfaces/etho/primary ip:  10.2.2.41

  (note no ENI card here as this will not have a  public ip as this is private)

* Security Group: New

  * Name : database_sg

  * Rules:

    *  Type: SSH / Protocol:  TCP /Port" 22/ Source: 192.168.0.0/16, 10.2.0.0/16
    *  Type: MYSQL/Aurora / Protocol:  TCP /Port 3306/ Source: 10.1.254.0/24
    *  Type: ALL ICMP-IPV4/ Protocol:  TCP /Port 0-65535/ Source: Anywhere

    





# VPC Peering & Routing

![](..\assets\aws-vpc-5.PNG)

Peering is only for instance to instance connections

Go to: VPC Dashboard/ Peering Connections

**Peering** 

Name: web-shared-pcx

VPC-Requester: web-vpc

VPC-Accepter:  shared-vpc

**Accept Request**  (on Peering window)

Actions:  Accept Request



**Set Routes**  (go to route tables)

routes for route table: web-pub 

Destination: 10.2.2.0/24 (database subnet)

Target:  web-shared-pcx



routes for route table: shared

Destination: 10.1.254.0/24

Target: web-shared-pcx

Instead of creating routes to subnets in VPC, create a subnet to the VPC

Destination:  10.1.0.0/16

Target: web-shared-pcx

```bash
ssh -i "pizza-keys.pem" ec2-user@34.206.56.131
ping 10.2.2.41


```

**If Peering does not work**

 [Peering Tutorial Quick start]( https://www.youtube.com/watch?v=okBWKQA_1Ws&feature=youtu.be ) 

1. In the instances Security Group, add an inbound rule

   Type: ALL ICMP-PV4

   Source: Anywhere

2. In instances Subnet, Check if route table is associated to the subnet



# Secure Ingress using a NAT Instance

  https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html 

You can use a network address translation (NAT) instance in a public subnet in your VPC to enable instances in the private subnet to initiate outbound IPv4 traffic to the Internet or other AWS services, but prevent the instances from receiving inbound traffic initiated by someone on the Internet. 

You can also use a NAT gateway (much more expensive), which is a managed NAT service that provides better availability, higher bandwidth, and requires less administrative effort. For common use cases, we recommend that you use a NAT gateway rather than a NAT instance. 



We are going to give database server outbound only (egress) access to the internet

![](..\assets\aws-vpc-6.PNG)

**Configuring a NAT Instance**

Here NAT1 instance (a linux vm) will have a route table pointing to the internet gateway (nat-pub route table)

1. Create a gateway, name : shared-igw and attach it to vpc: "shared-vpc"

2. Create nat-pub subnet in vnet: shared-vpc, in us-east-1a

   IPV4 Cidr: 10.2.254.0/24

3. Create a Route Table: 

    Name: nat-pub

   vpc: shared-vpc

4. Associate Subnet (Subnet Associations Tab)

   Edit subnet associations

   select nat-pub subnet

   Now you have an implied router

5. Add route for the internet gateway (Edit Routes)

   Destination: 0.0.0.0/0

   Target: internet gateway shared-igw

   

**Configuring a NAT Instance**

Amazon has provided a Linux image that is preconfigured to perform Network Address Translation

1. Allocate a new Elastic IP

2. Copy the PIP:  52.201.19.214

3. Go to EC2 Instances and launch a new instance

   1.  in Community AMIs, search ami-a7fdcadc
   2. Network: shared-vpc
   3. subnet: nat-pub
   4. Network interfaces/eth0/Primary IP/: 10.2.254.254

4. Security Group: create a new security group

   1.  Name:  NAT Instance

   2. Rule: 

      Type: SSH , Source: MyIP

       Type: ALL TCP, Source 10.0.0.0/8, 192.168.0.0/16,

      ​         description: VPC and on-prem networks

      (Note 10.0.0.0/8 covers both 10.1.0.0/16 and 10.2.0.0/16 ... I think; and we sonn't have an on-premise network yes)

   5. After instance is launched, go to instance, Actions and select "Networking/Change Source/Dest. Check"/ Yes Disable

      When db1 instance gets a  returns a message from the internet from the NAT, the NAT will use the source IP of the originating host  from internet. The Source/Destination IPs will differ and this check will block it. Hence we must disable this check

   6. Associate Elastic IP to the nat1 instance (Elastic IP/Instance=NAT Instance/Private IP =10.1.254.10)

## Configuring Routing for a NAT Client

Now we will configure the shared route table to point to the nat1 instance

Nat1 instance is associated with a public ip & can reach the internet

1. Go to shared route table
2. Add a route. Destination: 0.0.0.0/0, target: nat1 (select "instance", then nat1). Note that target changes to eni-...

## Using a Bastion Host (jump box)

You now use the nat1 instance as a jump box to configure the db1 instance

* ssh to nat1 instance using its elastic ip
* ssh to db1 instance using its private ip

```bash
#ssh into nat1 public ip
ssh -i "pizza-keys.pem" ec2-user@3.234.59.161

# copy key to db1 instance. First open pem file and copy in clipboard then paste on server
#ctrl-x to save

nano pizza-keys.pem

#give it restrictive permissions: remove read write; go=> group, other
cm

#ssh into db1
ssh -i "pizza-keys.pem" ec2-user@10.2.2.41
```



To prove that db1 can access internet

```bash
#on db1 instance ifconfig.co, which gets db1's ip, and you get back eip of nat1 instance 3.234.59.161
curl ifconfig.co
```

add WordPress on db1 (this will be our database) using docker

```bash
sudo docker run --name db1 -p 3306:3306 -d benpiper/aws-db1
```

add another WordPress instance on the www1 instance

```bash
ssh -i "pizza-keys.pem" ec2-user@34.206.56.131

sudo docker run --name www1 -p 80:80 -d benpiper/aws-www1
```



# Transit VPC

![](..\assets\aws-vpc-7.PNG)



## Create Transit VPC



## VPC

VPC Name: transit-vpc

VPC CIDR: 10.3.0.0/16

## Subnet

Subnet: 10.3.0.0/24

Subnet Name: transit

## Internet Gateway

Name: transit-igw

Attach to VPC:  transit-vpc

## Route Table

Name: transit

VPC: Transit

Route: 

	* Destination: 0.0.0.0/0
	* Target: transit_igw

Subnet Association:

​	Subnet:  transit

## Creating a CISCO CSR 1000V Instance

* Goto/ec2/launch instance/aws marketplace
* Search for car
* Select: **Cisco Cloud Services Router (CSR) 1000V - BYOL for Maximum Performance**
* Select t2.medium
* Network : transit-vpc
* Subnet: transit
* Network interfaces/eth0/primary ip: 10.3.0.10
* Security Group:  Rule allow ssh access from MyIP
* Associate a new public (elastic) ip to the instance: 52.86.229.56
* Ssh into Csr instance: ssh -i "hb.pem" ec2-user@52.86.229.56

## Creating a VPN Connection between two VPCs

VPN -> VPN Gateway -> VPN Connection->points to Cisco csr's elastic ip ->download configuration file->edit and replace csr's private ip->ssh into csr instance and add to configs

**Virtual Private Gateway in VPN to be connected**

1. Create a Virtual Private Gateway (goto: vis/Virtual private gateway): Name= shared-vgw
2. Attach to VPC: shared-vpc

**Create a VPN Connection**

Goto: VPN/Site-To-Site VPN Connections & click Create VPN Connection

* Name: transit-shared-vpn
* Virtual Private Gateway: shared-vgw
* Customer Gateway: New (refers to the transit csr instance we created above)
  * Ip: 52.86.229.56 (transit csr's elastic ip)
  * BGP ASN: 65000
  * Routing options: dynamic
  * Click create
* Download Configuration button click
  * Vendor:  Cisco Systems Inc
  * Platform: CSRv AMI
  * Software: ASA 9.7+ VTI
  * Search & Replace outside_interface with private ip of transit csr: 10.3.0.10
  * Copy entire file to clipboard
  * Ssh into transit csr:
    * conf t
    * paste
* Route propagation has to be enabled in the VPN with the VPN Gateway (in Routes Table): when enabled the route table will "learn" about routes from the csr instance

```bash
#to show learned routes: this will display route to other vpn--the share vp gateway adviterised this route via bgp
show ip route bgp

#ping db
ping db-ip source <csr internal ip>


```





## 