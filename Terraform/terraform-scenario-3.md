# **Scenario Description**

*Your project requires reusable infrastructure templates. Create a Terraform module for a web server setup and use it in multiple environments (dev and prod) by managing workspaces. Show how Terraform state files differ between these environments and implement basic security best practices like variable files for credentials.*

---

# **1. Folder Structure (Minimal)**

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

# **2. Terraform Module (Simple Web Server)**

**modules/web/main.tf**

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

# **3. Root Configuration**

**main.tf**

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

# **4. Environment Variable Files**

**dev.tfvars**

```hcl
instance_type = "t2.micro"
```

**prod.tfvars**

```hcl
instance_type = "t3.medium"
```

---

# **5. Initialize Terraform**

```bash
terraform init
```

> Downloads required providers and prepares Terraform to run.

---

# **6. Create Workspaces**

```bash
terraform workspace new dev
```

> Creates an isolated environment for dev with its own state file.

```bash
terraform workspace new prod
```

> Creates a separate environment for production infrastructure.

---

# **7. Select Workspace**

```bash
terraform workspace select dev
```

> Switches Terraform’s active environment to dev.

---

# **8. Apply Dev Environment**

```bash
terraform apply -var-file=dev.tfvars
```

> Deploys EC2 instance for dev using t2.micro instance type.

---

# **9. Switch to Prod**

```bash
terraform workspace select prod
```

> Changes active environment to production.

---

# **10. Apply Prod Environment**

```bash
terraform apply -var-file=prod.tfvars
```

> Deploys production instance (t3.medium) with separate state.

---

# **11. State File Separation**

Terraform automatically stores state as:

```
terraform.tfstate.d/dev/terraform.tfstate
terraform.tfstate.d/prod/terraform.tfstate
```

> Ensures dev and prod resources never interfere.

---

# Thank you

> Keep Exploring, Keep Automating 🚀

---
