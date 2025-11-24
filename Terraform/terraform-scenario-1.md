# Scenario Description

You need to manage multiple resources with dependencies. Create a Terraform configuration that deploys a web server and a security group, ensuring the server only launches after the security group is created. Demonstrate dependency management by reviewing the Terraform plan and output.

---

# 1. Folder Structure

```
terraform/
│── main.tf
│── variables.tf
│── outputs.tf
```

---

# 2. Terraform Configuration (Security Group + EC2)

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

# 3. Initialize Terraform

```bash
terraform init
```

> Downloads required providers and prepares Terraform to run.

---

# 4. Validate Configuration

```bash
terraform validate
```

> Checks Terraform syntax and structure.

---

# 5. Review Dependency Plan

```bash
terraform plan
```

> Shows that the security group is created before the EC2 instance because of implicit dependency.

---

# 6. Apply Infrastructure

```bash
terraform apply
```

> Creates the security group first, then the EC2 instance.

Type:

```
yes
```

> Confirms the creation process.

---

# 7. View Outputs

After apply:

```
public_ip = <public_ip_here>
sg_id     = <security_group_id_here>
```

> Displays the important resource values.

---

# 8. Destroy Infrastructure

```bash
terraform destroy
```

> Deletes EC2 instance first, then deletes the security group.

Type:

```
yes
```

> Confirms the destruction process.

---

# Thank you

> Keep Exploring, Keep Automating 🚀

---
