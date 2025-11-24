
# Scenario Description

You need to manage multiple resources with dependencies. Create a Terraform configuration that deploys a web server and a security group, ensuring the server only launches after the security group is created. Demonstrate dependency management by reviewing the Terraform plan and output.

---

# 1. Install Terraform (Windows and Ubuntu)

## Windows (PowerShell)

```powershell
choco install terraform -y
```

> Installs Terraform.

```powershell
terraform -v
```

> Verifies installation.

## Ubuntu

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

> Installs required dependencies.

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
```

> Downloads HashiCorp GPG key.

```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

> Adds Terraform repository.

```bash
sudo apt update
sudo apt install terraform
terraform -v
```

> Installs Terraform and verifies.

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

> Saves AWS access key, secret key, region, and output format.

Credentials stored at:

```
~/.aws/credentials
```

> Used automatically by Terraform for authentication.

---

# 4. Folder Structure

```
scenario-3/
│── main.tf
│── variables.tf
│── outputs.tf
```

---

# 5. Terraform Configuration

## main.tf

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
    Name = "Scenario3-Web-Server"
  }
}
```

---

# variables.tf

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

# outputs.tf

```hcl
output "public_ip" {
  value = aws_instance.web_server.public_ip
}

output "security_group_id" {
  value = aws_security_group.web_sg.id
}
```

---

# 6. Initialize Terraform

```bash
terraform init
```

> Downloads providers and prepares Terraform working directory.

---

# 7. Validate Configuration

```bash
terraform validate
```

> Ensures the configuration files are syntactically correct.

---

# 8. Review Dependency Plan

```bash
terraform plan
```

> Shows resource creation order, confirming that the security group is created before the EC2 instance because the EC2 resource references the security group ID.

---

# 9.Apply Infrastructure

```bash
terraform apply
```

> Executes the plan and creates resources.

Type:

```
yes
```

> Confirms deployment.

---

# 10. View Outputs

After apply:

```
public_ip           = <EC2 Public IP>
security_group_id   = <Security Group ID>
```

> Displays important resource identifiers.

---

# 11. Destroy Infrastructure

```bash
terraform destroy
```

> Removes both the EC2 instance and the security group in correct order.

Type:

```
yes
```

> Confirms deletion.
---


# Thank you

> Keep Exploring, Keep Automating 🚀

---
