
# Scenario Description

You need to manage multiple resources with dependencies. Create a Terraform configuration that deploys a web server and a security group, ensuring the server only launches after the security group is created. Demonstrate dependency management by reviewing the Terraform plan and output.

---

# 1. Install Terraform (Windows and Ubuntu)

## Windows (PowerShell)

```powershell
choco install terraform -y
```

> Installs Terraform using Chocolatey.

```powershell
terraform -v
```

> Verifies Terraform installation.

## Ubuntu

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

> Installs required packages.

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
```

> Downloads and configures HashiCorp GPG key.

```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

> Adds Terraform repository.

```bash
sudo apt update
sudo apt install terraform
terraform -v
```

> Installs and verifies Terraform.

---

# 2. Install AWS CLI (Windows and Ubuntu)

## Windows

```powershell
choco install awscli -y
```

> Installs AWS CLI.

```powershell
aws --version
```

> Confirms installation.

## Ubuntu

```bash
sudo apt install awscli -y
aws --version
```

> Installs and verifies AWS CLI.

---

# 3. Configure AWS Credentials

```bash
aws configure
```

> Saves access key, secret key, region, and output format for Terraform to use.

Credentials stored at:

```
~/.aws/credentials
```

> Automatically used by Terraform.

---

# 4. Folder Structure

```
terraform/
│── main.tf
│── variables.tf
│── outputs.tf
```

---

# 5. Terraform Configuration

### main.tf

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow HTTP"

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

resource "aws_instance" "web_server" {
  ami           = var.ami_id
  instance_type = var.instance_type

  vpc_security_group_ids = [aws_security_group.web_sg.id]

  tags = {
    Name = "Web-Server"
  }
}
```

---

### variables.tf

```hcl
variable "region" {
  default = "ap-south-1"
}

variable "ami_id" {
  default = "ami-0f58b397bc5c1f2e8"
}

variable "instance_type" {
  default = "t2.micro"
}
```

---

### outputs.tf

```hcl
output "public_ip" {
  value = aws_instance.web_server.public_ip
}

output "sg_id" {
  value = aws_security_group.web_sg.id
}
```

---

# 6. Initialize Terraform

```bash
terraform init
```

> Downloads AWS provider and prepares Terraform workspace.

---

# 7. Validate Configuration

```bash
terraform validate
```

> Checks that the configuration syntax is correct.

---

# 8. Review Dependency Plan

```bash
terraform plan
```

> Shows creation order: security group first, EC2 instance second.

---

# 9. Apply Infrastructure

```bash
terraform apply
```

> Creates both resources, enforcing dependency based on reference.

Type:

```
yes
```

> Confirms creation.

---

# 10. View Outputs

After apply:

```
public_ip = <EC2 Public IP>
sg_id     = <Security Group ID>
```

> Displays important details of created resources.

---

# 11. Destroy Infrastructure

```bash
terraform destroy
```

> Removes EC2 instance first, then the security group.

Type:

```
yes
```

> Confirms destruction.

---

# Thank you

> Keep Exploring, Keep Automating 🚀

---
