# Terraform AWS RHEL EC2 Instance

Deploying a RHEL EC2 Instance in AWS using Terraform

To update the version of RHEL, just update the ami line in the **linux-vm-main.tf** file, with a variable from the **rhel-versions.tf** file.

In this file, we support latest versions of:

- RHEL 7.x (7.7, 7.8 and 7.9)
- RHEL 8.x (8.3, 8.4 and 8.5)

# This repo has example code for the following:

## app-variables.tf
- 
## aws-user-data.sh
- initial startup script - for example;
```
#! /bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```
## key-pair-main.tf
- key pair for SSH - only for demonstration purposes
```# Generates a secure private key and encodes it as PEM
resource "tls_private_key" "key_pair" {
  algorithm = "RSA"
  rsa_bits  = 4096
}
# Create the Key Pair
resource "aws_key_pair" "key_pair" {
  key_name   = "${lower(var.app_name)}-${lower(var.app_environment)}-linux-${lower(var.aws_region)}"  
  public_key = tls_private_key.key_pair.public_key_openssh
}
# Save file
resource "local_file" "ssh_key" {
  filename = "${aws_key_pair.key_pair.key_name}.pem"
  content  = tls_private_key.key_pair.private_key_pem
}
```

## linux-vm-main.tf
- security groups, disks etc...
## linux-vm-output.tf
- dns, ip
  ```
  output "vm_linux_server_instance_id" {
  value = aws_instance.linux-server.id
}

output "vm_linux_server_instance_public_dns" {
  value = aws_instance.linux-server.public_dns
}

output "vm_linux_server_instance_public_ip" {
  value = aws_eip.linux-eip.public_ip
}

output "vm_linux_server_instance_private_ip" {
  value = aws_instance.linux-server.private_ip
}
```
## linux-vm-variables.tf
- Options for EC2 Virtual Machine creation
```variable "linux_instance_type" {
  type        = string
  description = "EC2 instance type for Linux Server"
  default     = "t2.micro"
}

variable "linux_associate_public_ip_address" {
  type        = bool
  description = "Associate a public IP address to the EC2 instance"
  default     = true
}

variable "linux_root_volume_size" {
  type        = number
  description = "Volumen size of root volumen of Linux Server"
}

variable "linux_data_volume_size" {
  type        = number
  description = "Volumen size of data volumen of Linux Server"
}

variable "linux_root_volume_type" {
  type        = string
  description = "Volumen type of root volumen of Linux Server. Can be standard, gp3, gp2, io1, sc1 or st1"
  default     = "gp2"
}

variable "linux_data_volume_type" {
  type        = string
  description = "Volumen type of data volumen of Linux Server. Can be standard, gp3, gp2, io1, sc1 or st1"
  default     = "gp2"
}
```
## network-main.tf
- VPC creation and updates
  
## network-variables.tf
- region and IPv4 Subnets - need to add IPv6
```

# AWS AZ
variable "aws_az" {
  type        = string
  description = "AWS AZ"
  default     = "eu-west-1c"
}

# VPC Variables
variable "vpc_cidr" {
  type        = string
  description = "CIDR for the VPC"
  default     = "10.1.64.0/18"
}

# Subnet Variables
variable "public_subnet_cidr" {
  type        = string
  description = "CIDR for the public subnet"
  default     = "10.1.64.0/24"
}
```
## provider-main.tf
- AWS info
```
provider "aws" {
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
  region     = var.aws_region
}
```
## provider-variables.tf
- AWS variables
  
## rhel-versions.tf
- Red Hat info
```

data "aws_ami" "rhel_8_5" {
  most_recent = true

  owners = ["309956199498"] // Red Hat's Account ID

  filter {
    name   = "name"
    values = ["RHEL-8.5*"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```
## terraform.tfvars
- Is this used or needed anymore?
```
# Application Definition 
app_name        = "kopicloud" # Do NOT enter any spaces
app_environment = "dev"       # Dev, Test, Staging, Prod, etc

# Network
vpc_cidr           = "10.11.0.0/16"
public_subnet_cidr = "10.11.1.0/24"

# AWS Settings
aws_access_key = "complete-this"
aws_secret_key = "complete-this"
aws_region     = "eu-west-1"

# Linux Virtual Machine
linux_instance_type               = "t2.micro"
linux_associate_public_ip_address = true
linux_root_volume_size            = 20
linux_root_volume_type            = "gp2"
linux_data_volume_size            = 10
linux_data_volume_type            = "gp2"
```
