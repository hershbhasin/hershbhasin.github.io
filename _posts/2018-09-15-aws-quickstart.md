---
layout: post
title: "AWS Quickstart"
author: "Hersh Bhasin"
comments: true
categories: AWS
published: false
---
Set up Local Environment with Access Key & Secret

```powershell
# you will be prompted for access key & secret
aws configure

# test
aws ec2 describe-instances

```



# VPC  (Virtual Private Cloud)

Equivalent to Azure Vnet. VPC is free.

1. Security Groups 

2. Each VPC has 1 Routing Table
3. Network Access Control List

![](..\assets\aws1.PNG)



Using Putty to ssh:

https://linuxacademy.com/guide/17385-use-putty-to-access-ec2-linux-instances-via-ssh-from-windows/