# Building AWS RDS Aurora Cluster with Multi Zone Availability

This reference architecture provides a set of YAML templates for deploying the following AWS services :
- Amazon IAM
- Amazon S3
- Amazon VPC
- Amazon NACL

## Prerequisites Notes
The Cloudformation Security Group IP address is open by default (testing purpose). You should update the Security Group Access with your own IP Address to ensure your instances security.

Before you can deploy this process, you need the following:
 - Your AWS account must have one VPC available to be created in the selected region
 - Amazon EC2 key pair

## Tested on the following Region:
 - US East (N. Virginia)

This template describes a VPC with two private and two public subnets.

![infrastructure-overview](images/Multi_AZ_with_RDS_Aurora_Cluster.png)

### Installation Guide
1. [![Launch Stack](https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=devRDS&templateURL=https://s3.amazonaws.com/cf-templates-hyv79l0oex7c-us-east-1/cloudformation-rds/master.yaml)
2. Click Next to proceed with the next step of the wizard.
3. Specify a name and all parameters for the stack.
4. Click Next to proceed with the next step of the wizard.
5. Click Next to skip the Options step of the wizard.
6. Click Create to start the creation of the stack.
7. Wait until the stack reaches the state CREATE_COMPLETE

The repository consists of a set of nested templates that deploy the following:

 - A tiered [VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html) with public and private subnets, spanning an AWS region.
 - A self-healing arc deployed across multi [Availability Zones](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) in an [Auto Scaling](https://aws.amazon.com/autoscaling/) group.
 - Two [NAT gateways](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html) to handle outbound traffic.

#### Infrastructure-as-Code

A template can be used repeatedly to create identical copies of the same stack (or to use as a foundation to start a new stack).  Templates are simple YAML- or JSON-formatted text files that can be placed under your normal source control mechanisms, stored in private or public locations such as Amazon S3, and exchanged via email. With CloudFormation, you can see exactly which AWS resources make up a stack. You retain full control and have the ability to modify any of the AWS resources created as part of a stack. 

#### Self-documenting 

Fed up with outdated documentation on your infrastructure or environments? Still keep manual documentation of IP ranges, security group rules, etc.?

With CloudFormation, your template becomes your documentation. Want to see exactly what you have deployed? Just look at your template. If you keep it in source control, then you can also look back at exactly which changes were made and by whom.

#### Intelligent updating & rollback

CloudFormation not only handles the initial deployment of your infrastructure and environments, but it can also manage the whole lifecycle, including future updates. During updates, you have fine-grained control and visibility over how changes are applied, using functionality such as [change sets](https://aws.amazon.com/blogs/aws/new-change-sets-for-aws-cloudformation/), [rolling update policies](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) and [stack policies](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/protect-stack-resources.html).

## Template details

The templates below are included in this repository and reference architecture:

| Template | Description |
| --- | --- | 
| [master.yaml](master.yaml) | This is the master template - deploy it to CloudFormation and it includes all of the nested templates automatically. |
| [infrastructure/rds-securitygroup.yaml](infrastructure/rds-securitygroup.yaml) | This template contains the [security groups](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html) required by the entire stack. |
| [infrastructure/rds-host.yaml](infrastructure/rds-host.yaml) | This template deploys a (Mysql Aurora) Relational Database Service. |
| [infrastructure/rds-route53.yaml](infrastructure/rds-route53.yaml) | This template deploys Route 53 recordset to update RDS Cluster Alias with CNAME entry. |

After the CloudFormation templates have been deployed, the [stack outputs](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html) contain a link to the site URLs.

![stack-outputs](images/rds_cloudformation_output.png)

![rds_cluster_1](images/rds_cluster_1.png)

![rds_cluster_2](images/rds_cluster_2.png)

![rds_instance_1](images/rds_instance_1.png)

![rds_instance_2](images/rds_instance_2.png)

![rds_instance_3](images/rds_instance_3.png)

![rds_instance_4](images/rds_instance_4.png)

## How do I...?

### Get started and deploy this into my AWS account

You can launch this CloudFormation stack in your account:

Example using AWS CLI Command :

1. First Download this code into your workstation, make your own changes and make the prerequisites updates.
 - Your AWS account must have one VPC available to be created in the selected region.
 - Create Amazon EC2 key pair
 - Install a domain in Route 53.
 - Install a certificate (in your selected region & also one in us-east-1) 

2. Next install [AWS CLI](aws.amazon.com/cli) in your workstation.

3. Upload the files in the "infrastructure" directory into to your own S3 bucket.
 - Eg. aws s3 cp --recursive infrastructure/ s3://cf-templates-19sg5y0d6d084-ap-southeast-1/

4. You can run the master.yaml file from your workstation.


```
Example Setup:

Run Time (~ 30 - 40 minutes)
===========================
To create a environment :
aws cloudformation create-stack \
--stack-name <env> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-rds//master.yaml

To update a environment :
aws cloudformation update-stack \
--stack-name <env> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-rds//master.yaml

To delete a environment :
aws cloudformation delete-stack --stack-name <env>

<env> - Note :stack-name that can be used are (devRDS, stagingRDS, prodRDS)

Example :
aws cloudformation create-stack \
--stack-name devRDS \
--capabilities=CAPABILITY_IAM \
--parameters ParameterKey=PMDBClusterIdentifier,ParameterValue=devcrytera \
--parameters ParameterKey=PMDBClusterParameterGroupName,ParameterValue=aurora-mysql5.7 \
--parameters ParameterKey=PMDatabaseEngine,ParameterValue=aurora-mysql \
--parameters ParameterKey=PMDatabaseEngineVer,ParameterValue=5.7.12 \
--parameters ParameterKey=PMDatabaseInstanceClass,ParameterValue=db.t2.small \
--parameters ParameterKey=PMDatabasePassword,ParameterValue=xxxxxx \
--parameters ParameterKey=PMDatabaseUsername,ParameterValue=ausertest1 \
--parameters ParameterKey=PMDomainSub,ParameterValue=myrds1 \
--parameters ParameterKey=PMDomainSubRO,ParameterValue=myrds1ro \
--parameters ParameterKey=PMEnvVPC,ParameterValue=devVPC \
--parameters ParameterKey=PMHostedZone,ParameterValue=crytera.com \
--parameters ParameterKey=PMKeyName,ParameterValue=mykey_nv \
--parameters ParameterKey=PMTemplateURL,ParameterValue=https://s3.amazonaws.com/cf-templates-hyv79l0oex7c-us-east-1/cloudformation-rds/infrastructure \
--template-body file:////path_to_template//cloudformation-rds//master.yaml
	
```

### Deploy multiple environments (e.g., dev, staging, production)

Deploy another CloudFormation stack from the same set of templates to create a new environment. The stack name provided when deploying the stack is prefixed to all taggable resources (e.g., EC2 instances, VPCs, etc.) so you can distinguish the different environment resources in the AWS Management Console. 

### Add a new item to this list

If you found yourself wishing this set of frequently asked questions had an answer for a particular problem, please [submit a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/). The chances are that others will also benefit from having the answer listed here.

## Contributing

Please [create a new GitHub issue](https://github.com/thinegan/cloudformation-rds/issues/new) for any feature requests, bugs, or documentation improvements. 

Where possible, please also [submit a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/) for the change. 

## Author

Thinegan Ratnam
 - [http://thinegan.com](http://thinegan.com/)

## Copyright and License

Copyright 2018 Thinegan Ratnam

Code released under the MIT License.