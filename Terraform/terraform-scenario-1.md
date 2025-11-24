
# Scenario Description

You are tasked with automating cloud resource creation. Install Terraform on your system, configure it to use AWS, and write a Terraform configuration file that provisions a single virtual machine instance. Use Terraform CLI commands to initialize, apply, and destroy infrastructure.

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

> Verifies Terraform installation.

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

> Adds Terraform repository to system.

```bash
sudo apt update
sudo apt install terraform
terraform -v
```

> Installs Terraform and verifies installation.

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

> Confirms the CLI installation.

## Ubuntu

```bash
sudo apt install awscli -y
aws --version
```

> Installs AWS CLI and verifies it.

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

> Used automatically by Terraform.

---

# 4. Folder Structure

```
terraform-vm/
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

resource "aws_instance" "vm" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "Single-VM-Demo"
  }
}
```

---

## variables.tf

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

## outputs.tf

```hcl
output "instance_id" {
  value = aws_instance.vm.id
}

output "public_ip" {
  value = aws_instance.vm.public_ip
}
```

---

# 6. Initialize Terraform

```bash
terraform init
```

> Downloads AWS provider and prepares Terraform working directory.

---

# 7. Validate Configuration

```bash
terraform validate
```

> Checks for syntax errors and structural issues.

---

# 8. Preview Execution

```bash
terraform plan
```

> Shows what Terraform will create and verifies configuration is correct.

---

# 9. Apply Configuration

```bash
terraform apply
```

> Creates the virtual machine on AWS.

Type:

```
yes
```

> Confirms resource creation.

---

# 10. View Outputs

After apply:

```
instance_id = <EC2_ID>
public_ip   = <PUBLIC_IP>
```

> Displays essential details of the created VM.

---

# 11. Destroy Infrastructure

```bash
terraform destroy
```

> Deletes the virtual machine from AWS.

Type:

```
yes
```

> Confirms deletion.

---
# Thank you

> Keep Exploring, Keep Automating 🚀

---
