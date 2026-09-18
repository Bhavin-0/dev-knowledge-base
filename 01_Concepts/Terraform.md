# Terraform

Use to create infrastucture to reduce the redudent work
such as creating configuration files (HCL format : Hashi corp lang)

##  What is terraform povider ?

A Terraform provider is a plugin that acts as a transaltor bridge between terraform and an external API, platform or service.

What providers do 
* Translate code : human readable -> specific API calls that targets platform understand
* Manage resources : CRUD operations items like servers, databases & networks 

----
NOTE : Terraform & provider verison (Maintained seperately), so make sure the version is compatible 

Review terraform documentatin as it changes frequently : https://registry.terraform.io/providers/hashicorp/aws/latest/docs
---

* while creating a new infrastructure (Terraform configuration block)

'''
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
}
'''

### What is CIDR block in vpc ?

In a VPC, CIDR (Classless Inter-Domain Routing) is a notation used to define the range of IP addresses available for you cloud 

## Terraform Commands
* terraform configure : to authenticate & autharize
* terraform plan : it calculates the delta between current state and desired state
* terraform apply: executes the action required to match the desired state / * terraform apply --auto-approve
* terraform destory: the resource will be destroyed 

----
* Instance types : are the build configurations of virtual servers designed with different resources

----
## Imp points to remember 
1. Store state file to remote backed
2. So not update/delete the file 
3. state locking 
4. Isolation of StateFile
5. Regular backup

## Key learnings with the documentaiton (https://developer.hashicorp.com/terraform/language/backend/s3)

1. storing state on S3 bucket supports state locking
2. Locking can be enabled using S3 & DynamoDB. But, DynamoDB is deprecated



