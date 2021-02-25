# Terraform

Terraform:
Terraform is an open source tool for building, changing, and versioning infrastructure safely and efficiently. Using Terraform we can automate provisioning of resources in cloud.

IAC ( Infrastructure as Code)
Physical organizational structures like Building, roads could be infrastructure. But in case of Software Development and operations an alternative of physical Infrastructure means underlying hardware we work with daily. Traditional physical infrasture like data center is expensive and hard to manage. An alternative of physical infrastructure is called Infrastructure as code.
HashiCorp Configuration Language (HCL) for human-readable, automated deployments.

Workflows:
1.	Scope - Confirm what resources need to be created for a given project.
2.	Author - Create the configuration file in HCL based on the scoped parameters
3.	Initialize: terraform init  => download the correct provider plug-ins for the project.
4.	Plan & Apply: terraform plan => to verify creation process. terraform apply => create real resources as well as state file. ( creates an infrastructure)

What is terraform in AWS?
Terraform by HashiCorp, is an “infrastructure as code” tool similar to AWS CloudFormation that allows you to create, update, and version your Amazon Web Services (AWS) infrastructure.

Is terraform better than CloudFormation?
CloudFormation covers most parts of AWS and needs some time to support new service capabilities. Terraform covers most AWS resources as well and is often faster than CloudFormation when it comes to supporting new AWS features. On top of that, Terraform supports other cloud providers as well as 3rd party services (docker.)


Ansible VS Terraform VS Puppet VS CHEF VS Saltstack


 
Why Terraform?
For EC2 instance launce, we need to configure Nat gateway/ Internet gateway, subnets, IAM role etc all of those activities manually we can complete automate the process using Terraform.

Syntax of writing Terraform
For each type of provider, there are many different kinds of resources that you can create, such as instrances, databases, security group, VPC and load balancers etc. Syntax for creating a resource in Terraform is:
 
Example:  
#Terraform setup
terraform {
  required_providers {
     aws = {
      source  = "hashicorp/aws"
      version = "3.23.0"
    }
  }
}
# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
  access_key = ""
  secret_key = ""
}

# Specify the provider and access details
Provider “aws”{				//Name of provider
	access_key = "${var.access_key}"
  secret_key = "${var.secret_key}"
Region 		= “us-west-2”		//region name

resource "aws_vpc" "main" {		//Type of resource 
  cidr_block       = "190.160.0.0/16"	//cidr id
  instance_tenancy = "default"

  tags = {
    Name = "main"
    Location = ”Banglore”
  }
}


=====================================================================



Resources:
https://www.youtube.com/watch?v=IxA1IPypzHs     
 [ how to create VPC and Subnets in aws using Terraform]
https://www.youtube.com/watch?v=vSKWBBrEbNQ
[how to create amazon machine image ]

https://medium.com/appgambit/terraform-aws-vpc-with-private-public-subnets-with-nat-4094ad2ab331
https://medium.com/faun/aws-s3-bucket-using-terraform-fb2e36c85d8a
https://medium.com/avmconsulting-blog/how-to-deploy-a-dockerised-node-js-application-on-aws-ecs-with-terraform-3e6bceb48785

Example:
/*==== The VPC ======*/
	resource "aws_vpc" "vpc" {
	  cidr_block           = "${var.vpc_cidr}"
	  enable_dns_hostnames = true
	  enable_dns_support   = true
	  tags = {
	    Name        = "${var.environment}-vpc"
	    Environment = "${var.environment}"
	  }
	}
	/*==== Subnets ======*/
	/* Internet gateway for the public subnet */
	resource "aws_internet_gateway" "ig" {
	  vpc_id = "${aws_vpc.vpc.id}"
	  tags = {
	    Name        = "${var.environment}-igw"
	    Environment = "${var.environment}"
	  }
	}
	/* Elastic IP for NAT */
	resource "aws_eip" "nat_eip" {
	  vpc        = true
	  depends_on = [aws_internet_gateway.ig]
	}
	/* NAT */
	resource "aws_nat_gateway" "nat" {
	  allocation_id = "${aws_eip.nat_eip.id}"
	  subnet_id     = "${element(aws_subnet.public_subnet.*.id, 0)}"
	  depends_on    = [aws_internet_gateway.ig]
	  tags = {
	    Name        = "nat"
	    Environment = "${var.environment}"
	  }
	}
	/* Public subnet */
	resource "aws_subnet" "public_subnet" {
	  vpc_id                  = "${aws_vpc.vpc.id}"
	  count                   = "${length(var.public_subnets_cidr)}"
	  cidr_block              = "${element(var.public_subnets_cidr,   count.index)}"
	  availability_zone       = "${element(var.availability_zones,   count.index)}"
	  map_public_ip_on_launch = true
	  tags = {
	    Name        = "${var.environment}-${element(var.availability_zones, count.index)}-      public-subnet"
	    Environment = "${var.environment}"
	  }
	}
	/* Private subnet */
	resource "aws_subnet" "private_subnet" {
	  vpc_id                  = "${aws_vpc.vpc.id}"
	  count                   = "${length(var.private_subnets_cidr)}"
	  cidr_block              = "${element(var.private_subnets_cidr, count.index)}"
	  availability_zone       = "${element(var.availability_zones,   count.index)}"
	  map_public_ip_on_launch = false
	  tags = {
	    Name        = "${var.environment}-${element(var.availability_zones, count.index)}-private-subnet"
	    Environment = "${var.environment}"
	  }
	}
	/* Routing table for private subnet */
	resource "aws_route_table" "private" {
	  vpc_id = "${aws_vpc.vpc.id}"
	  tags = {
	    Name        = "${var.environment}-private-route-table"
	    Environment = "${var.environment}"
	  }
	}
	/* Routing table for public subnet */
	resource "aws_route_table" "public" {
	  vpc_id = "${aws_vpc.vpc.id}"
	  tags = {
	    Name        = "${var.environment}-public-route-table"
	    Environment = "${var.environment}"
	  }
	}
	resource "aws_route" "public_internet_gateway" {
	  route_table_id         = "${aws_route_table.public.id}"
	  destination_cidr_block = "0.0.0.0/0"
	  gateway_id             = "${aws_internet_gateway.ig.id}"
	}
	resource "aws_route" "private_nat_gateway" {
	  route_table_id         = "${aws_route_table.private.id}"
	  destination_cidr_block = "0.0.0.0/0"
	  nat_gateway_id         = "${aws_nat_gateway.nat.id}"
	}
	/* Route table associations */
	resource "aws_route_table_association" "public" {
	  count          = "${length(var.public_subnets_cidr)}"
	  subnet_id      = "${element(aws_subnet.public_subnet.*.id, count.index)}"
	  route_table_id = "${aws_route_table.public.id}"
	}
	resource "aws_route_table_association" "private" {
	  count          = "${length(var.private_subnets_cidr)}"
	  subnet_id      = "${element(aws_subnet.private_subnet.*.id, count.index)}"
	  route_table_id = "${aws_route_table.private.id}"
	}
	/*==== VPC's Default Security Group ======*/
	resource "aws_security_group" "default" {
	  name        = "${var.environment}-default-sg"
	  description = "Default security group to allow inbound/outbound from the VPC"
	  vpc_id      = "${aws_vpc.vpc.id}"
	  depends_on  = [aws_vpc.vpc]
	  ingress {
	    from_port = "0"
	    to_port   = "0"
	    protocol  = "-1"
	    self      = true
	  }
	  
	  egress {
	    from_port = "0"
	    to_port   = "0"
	    protocol  = "-1"
	    self      = "true"
	  }
	  tags = {
	    Environment = "${var.environment}"
	  }
	}
Command: 
terraform plan
terraform init
terraform apply
terraform destroy


