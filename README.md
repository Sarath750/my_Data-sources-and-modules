# my_Data-sources-and-modules
Terraform data resources and modules
------------------------------------
Data resources
-------------
If there are any existing resources in aws, and if we want to use that same existing resources in our terraform scripts then we can use data resource.

create a security group named "ssh_sg" in ec2-instance manually. So, we can use this existing security group in our terraform script using data sources.

provider.tf
-----------
provider "aws" {
  region     = var.region
  version = "~> 4.0"
  access_key = "AKIAS5QJFNKWB2YIVC22"
  secret_key = "j8+fQUToRgqUbWwzup7JgPYlrWb2CFPWHWPCMij5"
} 


instance.tf
-----------
data "aws_key_pair" "example" {
   key_name     = "aws8am"
}

data "aws_security_group" "sg" {
  filter{
   name = "group-name"
   values = ["ssh_sg"]
  } 
}

resource "aws_instance" "web" {
  ami           = lookup(var.ami-id,var.region)
  instance_type = var.instance_type
  security_groups = [data.aws_security_group.sg.name]
  key_name   = data.aws_key_pair.example.key_name
  

  tags = {
    Name = var.instance_tag
  }
}

variable.tf
-----------
variable "ami-id" {
    type = map
    default = {
    "us-east-1" = "ami-090fa75af13c156b4"
    "us-west-2" = "ami-0cea098ed2ac54925"
  }
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "instance_tag" {
  type = string
  default = "HelloWorld"
}

variable "region" {
  type = string
}


When we give the commands terraform init, terraform plan, terraform apply our instance may not be creating because there may be one more policy  added in the iam user which we created, so delete that added policy then give these commands everything works absolutely fine.

Now, we have created ec2-instance with existing security-group(ssh_sg) and key-pair(aws8am).


terraform apply -var="region=us-east-1" -auto-approve
terraform destroy -var="region=us-east-1" -auto-approve





Modules
-------

In this module, skeleton structure will be available for instance.tf and variable.tf and provider.tf. so that anyone can use these modules for creating their own instances.





/c/Linux AWS DevOps/Terraform/modules
1. ec2-module
     aws-instance.tf
     variables.tf
2. security-group 
    aws-security-group.tf
    securitygroup-variable.tf

/c/Linux AWS DevOps/Terraform/re-use_modules
1. re-use_module
    ec2-instance-reusemodule.tf
    provider.tf
    securitygroup_reuse_module.tf
    


1. ec2-module

aws-instance.tf
---------------

resource "aws_instance" "web" {
  ami           = var.ami-id
  instance_type = var.instance_type 
  key_name   = var.key_name

  tags = {
    Name = var.instance_tag
  }
}

variables.tf
------------
variable "ami-id" {
    type = string
}

variable "instance_type" {
  type    = string
}

variable "instance_tag" {
  type = string
}

variable "region" {
  type = string
}

variable "key_name" {
  type    = string
}

Now we have provided the skeleton structure for ec2-module, and declared the variables.

2. security-group 

aws-security-group.tf
---------------------
resource "aws_security_group" "ssh_port" {
  name        = var.sg_name
  description = var.sg_name

  ingress {
    description      = var.sg_name
    from_port        = var.port
    to_port          = var.port
    protocol         = "tcp"
    cidr_blocks 	 = ["0.0.0.0/0"] 
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
  }

  tags = {
    Name = var.tagname
  }
}


securitygroup-variable.tf
-------------------------
variable "sg_name" {
  type = string
}

variable "port" {
  type = string
}

variable "tagname" {
  type = string
}


Now we have created the skeleton-structure for security-group and declared the variables as well.







/c/Linux AWS DevOps/Terraform/re-use_modules
1. re-use_module

ec2-instance-reusemodule.tf
---------------------------
module "example_instance" {
    source = "../modules/ec2-module"
    ami-id = "ami-090fa75af13c156b4" 
    instance_type = "t2.micro"
    instance_tag = "HelloWorld"
    region = "us-east-1"
    key_name = "8ambatch"

}


provider.tf
-----------
provider "aws" {
  region     = "us-east-1"
  access_key = "AKIAS5QJFNKWB2YIVC22"
  secret_key = "j8+fQUToRgqUbWwzup7JgPYlrWb2CFPWHWPCMij5"
} 

securitygroup_reuse_module.tf
-----------------------------
module "example_securitygroup" {
    source = "../modules/security_group"
    sg_name = "securitygroup_1"
    port = 22
    tagname = "securitytag"
}


Now we can create 2 resources(ec2-instance, securitygroup) using the skeleton-structure given above. here in the re-use modules we just need to provide the values to the skeleton structure.

