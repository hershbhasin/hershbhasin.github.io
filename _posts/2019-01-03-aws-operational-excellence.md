---
layout: post
title: "AWS Operational Excellence"
author: "Hersh Bhasin"
comments: true
categories: AWS
permalink: /aws-operational-excellence/
---



Study notes for course: [Architecting for Operational Excellence on AWS | Pluralsight](https://app.pluralsight.com/library/courses/architecting-operational-excellence-aws)

With multiple Cloudformation templates, we will build

![](..\assets\aws-ops-1.PNG)



# Create a new IAM User

1. under IAM Users create a new user

2. Grant Programmatic access

3. Attach existing policies directly:  AdministratorAccess

4. Note Access Key Id & access Key

5. In terminal run command below & enter credentials & 

   1. region (us-east-1)
   2. default output format: json

   ``` bash
   aws configure
   
	#check
	aws iam list-users
	```



# Deploy stack containing VPC and 2 public subnets

Template Location: [https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc.yaml]( https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc.yaml)



```bash
#Create Stack
aws cloudformation create-stack --stack-name vpc-production --template-url https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc.yaml

#Check stack progress
aws cloudformation describe-stacks

#check stack
aws cloudformation list-stack-resources --stack-name vpc-production

#Delete a stack
aws cloudformation delete-stack --stack-name vpc-production

# pass parameters
aws cloudformation create-stack --stack-name vpc-production --template-url  https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc.yaml --parameters ParameterKey=VpcCIDR,ParameterValue=10.8.0.0/22  ParameterKey=PublicSubnet1CIDR,ParameterValue=10.8.1.0/24 ParameterKey=PublicSubnet2CIDR,ParameterValue=10.8.2.0/24

#Updating stacks
aws cloudformation update-stack --stack-name vpc-production --template-url  https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc.yaml --parameters ParameterKey=VpcCIDR,ParameterValue=10.7.0.0/22  ParameterKey=PublicSubnet1CIDR,ParameterValue=10.7.1.0/24 ParameterKey=PublicSubnet2CIDR,ParameterValue=10.7.2.0/24

#Check status of update
aws cloudformation list-stacks --stack-status-filter UPDATE_COMPLETE

```

### Update Behaviors

1. Update with no interruption (physical id not changed)
2. Update with some interruption (physical id not changed, but interruption occurs, say rebooting ec2 instance)
3. Replacement (id changes -- vpc CIDR changes require replacement: rip & replace)



# Advanced CloudFormation Concepts

## Stack Policies

Applies to stack updates. Controls weather a stack update can modify replace or delete resources. Does not prevent manual modification. Can be only applied upon stack creation.

Policy location:   [https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc-policy.json](https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc-policy.json)

The following rule initially allows all updates, then denies update & replace to a VPC

```json
{
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "*"
    },
    {
      "Effect" : "Deny",
      "Action" : "Update:Replace",
      "Principal": "*",
      "Resource" : "LogicalResourceId/VPC",
      "Condition" : {
        "StringLike" : {
            "ResourceType" : ["AWS::EC2::VPC"]
        }
      }
    }
  ]
}
```

Since Stack policy can only be applied at stack creation time, we must delete existing vpc & recreate with policy

```bash
#delete
aws cloudformation delete-stack --stack-name vpc-production

#recreate with policy

aws cloudformation create-stack --stack-name vpc-production --template-url  https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc.yaml --stack-policy-url https://s3.amazonaws.com/architecting-operational-excellence-aws/vpc-policy.json
```

Stack policies cannot de deleted but you can update with a updated policy using the *stack-policy-during-update-url* flag. The stack policy will be overridden only for that update.

![](..\assets\aws-ops-2.PNG)



# Stack Outputs

Can be accessed via the DescribeStacks API call

Export Name & Exportvalue can be accessed from the CloudFormation/outputs tab  and the Exports navigation on left. You use the Export Name to ImportValues in other stacks (next section)

# Create a Load Balancer with Stack Export Values

Template location:   [https://s3.amazonaws.com/architecting-operational-excellence-aws/load-balancer.yaml](https://s3.amazonaws.com/architecting-operational-excellence-aws/load-balancer.yaml)

```yaml
Fn::ImportValue:
          !Sub "${VPCStackName}-PublicSubnet1"
```



Here ImportValue is  a function that imports a exported name from another stack, and the Sub function substitutes it.  The VPCStackName is a parameter that gets passed into this stack, which is the name of the parent stack, which in this case is called vpc-Production. The stack vpc-production exports the subnet name as  vpc-production-PublicSubnet1.

```yaml
 ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Subnets:
      - Fn::ImportValue:
          !Sub "${VPCStackName}-PublicSubnet1"
      - Fn::ImportValue:
          !Sub "${VPCStackName}-PublicSubnet2"
      SecurityGroups:
      - Ref: ALBSecurityGroup
```



Create the load balancer

```bash
aws cloudformation create-stack --stack-name alb-production --template-url  https://s3.amazonaws.com/architecting-operational-excellence-aws/load-balancer.yaml --parameters ParameterKey=VPCStackName,ParameterValue=vpc-production
```

![](..\assets\aws-ops-3.PNG)

# Nested Stacks

Delete the Load Balancer stack: alb-production. We will create it as a nested stack.

A nested stack is created by a parent stack. A new Auto scaling parent stack will create a nested load balancer stack.

Template location:   https://s3.amazonaws.com/architecting-operational-excellence-aws/auto-scaling.yaml

![](..\assets\aws-ops-4.PNG)



# Notes

Ref: & !Ref  mean the same. !Ref is the short form and can only be used in yaml format.

# References

[Cloud Formation Intrensic Function References](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)