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

