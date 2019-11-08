---
layout: post
title: "AWS Operational Excellence"
author: "Hersh Bhasin"
comments: true
categories: AWS
permalink: /aws-operational-excellence/
---

In this post, we will look at 

1. Relationship between stacks and templates
2. Use parameters to pass values to stacks
3. Understand stack update behaviors and policies
4. Use stack outputs

This post is is essentially my  study notes for the excellent Pluralsight course: [Architecting for Operational Excellence on AWS | Pluralsight](https://app.pluralsight.com/library/courses/architecting-operational-excellence-aws) by Ben Piper

With multiple CloudFormation templates, we will build the following:

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

The InstanceRole is the identity & access management role that the instance will assume.

The Userdata section runs scripts on startup of each instance. It looks like this

```yaml
UserData:
          Fn::Base64:
            Fn::Sub: |
              #!/bin/bash -xe
              sudo yum -y update
              sudo yum -y install aws-cfn-bootstrap
              /opt/aws/bin/cfn-signal -e 0 --region ${AWS::Region} --stack ${AWS::StackName} --resource AutoScalingGroup
```

It instructs each instance to install the CloudFormation bootstrap package (aws-cfn-bootstrap). In that package is a utility cfn-signal (CloudFormation Signal) which signals CloudFormation when each instance is up and running. This is a feedback mechanism.



The  AutoScaling Group within the template has a section called CreationPolicy

```yaml
CreationPolicy:
      ResourceSignal:
        Timeout: PT10M
        Count:
          Ref: GroupSize
```

The Timeout means pause for 10 minutes (PT=Pause Time; PT10M= pause for 10 minutes) after stack is created. Then Count for the GroupSize. The GroupSize is a passed parameter (with default of 2), which specifies the number of instances to create. If within a period of 10 minutes, CloudFormation does not get 2 signals, it will declare the stack creation a failure.

This is how the AutoScalingGroup looks in the template

```yaml
 AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      VPCZoneIdentifier:
      - Fn::ImportValue:
          Fn::Sub: "${VPCStackName}-PublicSubnet1"
      - Fn::ImportValue:
          Fn::Sub: "${VPCStackName}-PublicSubnet2"
      LaunchTemplate:
        LaunchTemplateId:
          Ref: WebserverLaunchTemplate
        Version: '1'
      MinSize: '1'
      MaxSize: '6'
      DesiredCapacity:
        Ref: GroupSize
      TargetGroupARNs:
      -  Fn::GetAtt: [ ALBStack, Outputs.ALBTargetGroup ]
    CreationPolicy:
      ResourceSignal:
        Timeout: PT10M
        Count:
          Ref: GroupSize
```



Bring up the stack

```bash
 aws cloudformation create-stack --stack-name auto-scaling-production --template-url   https://s3.amazonaws.com/architecting-operational-excellence-aws/auto-scaling.yaml --parameters ParameterKey=VPCStackName,ParameterValue=vpc-production  ParameterKey=KeyName,ParameterValue=pizza-keys --capabilities CAPABILITY_NAMED_IAM
```



Notice in script above: *--capabilities CAPABILITY_NAMED_IAM*

Without this flag, will give a capability exception: CAPABILITY_NAMED_IAM error as we are creating a IAM role in the stack. We must explicitly allow this, hence the flag.



# Code Commit (Git)

## Configure the developer user

1. Long into the AWS Management Console as Administrator
2. Generate CodeCommit Credentials for IAM user (this is the user we created in first section"Create a new IAM User")
   1. IAM/User/Security credentials/console password/click manage/check enable, set custom password
   2. Scroll to section "Https Git credentials for AWS CodeCommit", Generate & download credentials
   3. Log into console as IAM user: Scroll up  to "Sign-in credentials" and next to Summary, click on the link: console sign-in link, enter the user name & password for the iam user

# Code Deploy

Hooks:  https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html 

CodeDeploy requries a appspec.yml  file that lists out the steps 

1. First ApplicationStop fires
2. Before Install
3. Then Files (means: install)
4. AfterInstall

```yaml
version: 0.0
os: linux
files:
  - source: index.html
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 180
      runas: root
  AfterInstall:
    - location: scripts/change_index.sh
      timeout: 30
      runas: root
    - location: scripts/start_server.sh
      timeout: 60
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 60
      runas: root

```

## Installing CodeDeploy Agent

Agent must be running on EC2 instances. Use the AWS Systems Manager to push CodeDeploy agent to EC2 instances.

Create a command document (installCodeDeployAgent.yml). Region might need to be cahanged



installCodeDeployAgent.yml

```yaml
---
schemaVersion: '2.2'
description: cross-platform sample
mainSteps:
- action: aws:runShellScript
  name: InstallCodeDeployAgent
  precondition:
    StringEquals:
    - platformType
    - Linux
  inputs:
    runCommand:
    - yum install -y ruby
    - yum install -y aws-cli
    - cd /home/ec2-user
    - aws s3 cp s3://aws-codedeploy-us-east-1/latest/install . --region us-east-1
    - chmod +x ./install
    - ./install auto

```



Note. Agent could have been installed when provisioning the EC2 instance by adding this in the user data section

```yaml
- yum install -y ruby
    - yum install -y aws-cli
    - cd /home/ec2-user
    - aws s3 cp s3://aws-codedeploy-us-east-1/latest/install . --region us-east-1
    - chmod +x ./install
    - ./install auto
```





Go to AWS System Manager:  https://console.aws.amazon.com/systems-manager/home?region=us-east-1# 

1. Under navigation link "documents", create a document
2. Document Type: Command document
3. Paste above Yaml file
4. The document will appear in the owned by me tab



![](..\assets\aws-ops-5.PNG)



## Run the Document

1. Under Instance & Nodes, click link RunCommand
2. Click on orange button RunCommand
3. Search for Owner/Owned by me
4. Select the Document
5. Under Targets, Select "Choose Instances Manually"
6. Choose the instances
7. Scroll to Output Options, uncheck "Enable writing to S3 bucket"
8. Scroll down and click Run

##  Log into the EC2 instance to verify agent

Instead of using ssh, we will use System Manager's "Session Manager" link -- it is in the Instance & Nodes tab, above Run Command

1. Click on Session Manager link
2. Click button (orange) Start Session
3. Select an instance
4. Check status of agent by running command below

```bash
sudo service codedeploy-agent status
```

## Create an IAM service role for Code Deploy

1. Go To IAM roles
2. Click Create Role
3. Click on link "CodeDeploy"
4. Select your use case : CodeDeploy (the first)
5. accept   [AWSCodeDeployRole](https://console.aws.amazon.com/iam/home?region=us-east-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2Fservice-role%2FAWSCodeDeployRole) 
6. Name it: AWSCodeDeployRole
7. Click Create Role

## Create CodeDeploy Application

1. Go to CodeDeploy console
2. Click on Deploy/Applications nav link
3. Application Configuration
   1. Application Name : sampleapp
   2. Compute platform : Ec2/On-Premises

## Create Deployment Group

Deployment group specifies which instances to deploy to ( we are deploying to a autoscaling group)

1. Click on Create Deployment Group
2. Deployment Grout Name :  sampleapp-asg
3. Service Role:  select the role created above -- AWSCodeDeployRole
4. Deployment Type : In-place
5. Environment Configuration: Amazon EC2 Auto Scaling Group
   * Auto Scaling Group: select your auto scaling group
6.  Deployment Settings: CodeDeployDefault.AllAtOnce
7. Load Balancer: choose your lb
8. Create Deployment Group

## Creating a Deployment

Specifying the location of the application files. You cannot deploy an application directly from a CodeCommit repo. by using CodeDeploy

Deploy the sample application from an s3 bucket.

1. zip the code file and upload it to S3
2. Go to CodeDeploy/Applications, select your Deployment Group
3. click Create Deployment
4. specify s3 path
5. Create deployment

# CodePipeline

Configure CodePipeline to deploy sample application from CodeCommit repository

1. Source Stage: CodeCommit
2. Deployment stage: Code Deploy

Procedure

1. Go to CodePipeline service console
2. Click: Create Pipeline
3. Name: sampleapp-ce
4. New service role checkbox: checked
5.   Allow AWS CodePipeline to create a service role ... checkbox: checked
6. Advanced settings/Artifact store/Default location: checked (CodePipeline will copy files from your CodeCommit repo and store it in s3 as a zip)
7. Source: AWSCodeCommit -- then give details about your git repo and branch
8. Build stage: skip, as our app is a simple html page with nothing to build
9. Deploy:  AWS CodeDeploy, select Application name & Deployment group
10.  Click: Create Pipeline



# Notes

Ref: & !Ref  mean the same. !Ref is the short form and can only be used in yaml format.

# References

[Cloud Formation Intrensic Function References](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

 [Architecting for Operational Excellence on AWS | Pluralsight](https://app.pluralsight.com/library/courses/architecting-operational-excellence-aws)