## Terraform AWS
As of my last update, Ubuntu 24.04 and ERPNext Version 15 have not been officially released, so the workflow can be subject to changes once they are available. However, for the sake of an example, I’ll provide you steps to automate the installation on a current version of Ubuntu (e.g., 22.04) using Terraform.

To give context, Terraform is used for infrastructure as code to manage diverse infrastructural components. In this case, we can use it to create an instance and install ERPNext.

### Pre-requisites
1. **Install Terraform** on your local machine.
    - Follow instructions from [HashiCorp's website](https://learn.hashicorp.com/tutorials/terraform/install-cli).
2. **Set up cloud service provider access** (AWS, Azure, GCP, etc.).
    - This example will use AWS EC2. Set up your IAM roles and keys appropriately.
3. **Ensure the Frappe Docker to install ERPNext**.

### Step By Step Guide

#### 1. Create Terraform configuration files

Firstly, let's set up your Terraform files to create an infrastructure suitable for installing ERPNext.

##### main.tf
Create a file called `main.tf` and add the following content:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "erpnext" {
  ami           = "ami-xxxxxx"  # Replace with Ubuntu 22.04 AMI ID from your region
  instance_type = "t2.medium"   # Use an appropriate instance type

  # Add your keypair name
  key_name = "your-key-pair"

  # Configure the security group for this instance
  vpc_security_group_ids = [aws_security_group.erpnext-sg.id]

  # User data to configure the instance at boot time
  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get upgrade -y
              apt-get install -y \
                ca-certificates \
                curl \
                gnupg \
                lsb-release
                
              curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
              echo \
                "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
                $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
                
              apt-get update -y
              apt-get install -y docker-ce docker-ce-cli containerd.io

              sudo systemctl start docker
              sudo systemctl enable docker

              docker run -d --name erpnext --restart always -p 80:80 frappe/erpnext:version-13 # Change this to version 15 once it's available
              EOF

  tags = {
    Name = "ERPNext-Instance"
  }
}

resource "aws_security_group" "erpnext-sg" {
  name_prefix = "erpnext-sg"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

##### variables.tf
If you would like to make the configuration more flexible, you can define some variables:

```hcl
variable "region" {
  description = "AWS region for the deployment"
  default     = "us-east-1"
}

variable "instance_type" {
  description = "Type of instance to be created"
  default     = "t2.medium"
}

variable "key_name" {
  description = "Name of the key pair"
  default     = "your-key-pair"
}
```

##### outputs.tf
To output information as needed:

```hcl
output "instance_id" {
  description = "ID of the created instance"
  value       = aws_instance.erpnext.id
}

output "public_ip" {
  description = "Public IP of the created instance"
  value       = aws_instance.erpnext.public_ip
}

output "public_dns" {
  description = "Public DNS of the created instance"
  value       = aws_instance.erpnext.public_dns
}
```

#### 2. Deploy with Terraform

After you've written your Terraform configurations, use the following commands to deploy the instance and set up ERPNext:

```sh
terraform init
terraform plan
terraform apply
```

When prompted, type `yes` to confirm the deployment.

#### 3. Post-Deployment

- Connect to your EC2 instance via SSH:

```sh
ssh -i "path-to-your-keypair.pem" ubuntu@your-instance-public-ip
```

- Verify Docker containers:

```sh
docker ps
```

- Open your instance's public IP in a web browser to access ERPNext.

Please replace the placeholders like `your-key-pair`, `us-east-1`, `ami-xxxxxx` with your actual configurations. Once ERPNext Version 15 and Ubuntu 24.04 are released, you will need to update the Docker image tag and AMI to match the new versions.

### Final Note

Always ensure you're keeping up to date with the latest documentation for Terraform, AWS, Docker, and ERPNext since APIs and best practices can evolve.



