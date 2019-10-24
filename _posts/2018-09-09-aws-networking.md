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

# web-vpc

**Public VPC**

Name: web-vpc

CIDR: 10.1.0.0/16

**Public subnet**

Name: web-pub

CIDR: 10.1.254.0/24

Zone: us-east-1a

**Route Table** (aka "Implied Router")

Name: web-pub

Associated subnet: web-pub (10.1.254.0/24)



Each VPC has a default route table which is automatically associated with each subnet you create.

All traffic into, out of and within a vpc traverses a implied router. It is the gatekeeper and the brains of vpc networking