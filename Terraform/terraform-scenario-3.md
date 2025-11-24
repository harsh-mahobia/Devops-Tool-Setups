
# Scenario Description

Your project requires reusable infrastructure templates. Create a Terraform module for a web server setup and use it in multiple environments (dev and prod) by managing workspaces. Show how Terraform state files differ between these environments and implement basic security best practices like variable files for credentials.

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

> Confirms the installation.

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

> Stores access key, secret key, region, and output format.

Credentials saved at:

```
~/.aws/credentials
```

> Used automatically by Terraform.

---

# 4. Folder Structure (Minimal and Clean)

```
terraform/
│── main.tf
│── dev.tfvars
│── prod.tfvars
│── modules/
      └── web/
          └── main.tf
```

---

# 5. Terraform Module

### modules/web/main.tf

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "${var.env}-web"
  }
}

variable "ami" {}
variable "instance_type" {}
variable "env" {}
```

---

# 6. Root Terraform Configuration

### main.tf

```hcl
provider "aws" {
  region = "ap-south-1"
}

module "webserver" {
  source        = "./modules/web"
  ami           = var.ami
  instance_type = var.instance_type
  env           = terraform.workspace
}

variable "ami" {
  default = "ami-0f58b397bc5c1f2e8"
}

variable "instance_type" {}
```

---

# 7. Environment Variable Files

### dev.tfvars

```hcl
instance_type = "t2.micro"
```

### prod.tfvars

```hcl
instance_type = "t3.medium"
```

---

# 8. Initialize Terraform

```bash
terraform init
```

> Downloads AWS provider and prepares the working directory.

---

# 9. Create Workspaces

```bash
terraform workspace new dev
```

> Creates isolated dev environment.

```bash
terraform workspace new prod
```

> Creates isolated prod environment.

---

# 10. Select Workspace

```bash
terraform workspace select dev
```

> Switches the working environment to dev.

---

# 11. Apply Dev Environment

```bash
terraform apply -var-file=dev.tfvars
```

> Deploys the dev EC2 instance using module and dev variables.

Type:

```
yes
```

> Confirms the deployment.

---

# 12. Switch to Prod

```bash
terraform workspace select prod
```

> Activates production environment.

---

# 13. Apply Prod Environment

```bash
terraform apply -var-file=prod.tfvars
```

> Deploys production EC2 instance with its own configuration.

Type:

```
yes
```

> Confirms the deployment.

---

# 14. State File Separation

Terraform automatically creates workspace-specific state:

```
terraform.tfstate.d/dev/terraform.tfstate
terraform.tfstate.d/prod/terraform.tfstate
```

> Keeps dev and prod resources completely isolated.

---

# 15. Destroy Resources When Needed

```bash
terraform destroy -var-file=dev.tfvars
```

> Deletes resources in the dev workspace.

```bash
terraform workspace select prod
terraform destroy -var-file=prod.tfvars
```

> Deletes production resources safely.

---

# Thank you

> Keep Exploring, Keep Automating 🚀

---
