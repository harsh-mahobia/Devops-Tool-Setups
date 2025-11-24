# **Scenario Description**

> *You are tasked with automating cloud resource creation. Install Terraform on your system, configure it to use AWS, and write a Terraform configuration file that provisions a single virtual machine instance. Use Terraform CLI commands to initialize, apply, and destroy infrastructure.*

---

# **1. Install Terraform (Windows + Ubuntu)**

## **Windows**

### **Using Chocolatey**

```powershell
choco install terraform -y
```

### **Verify**

```powershell
terraform -v
```

---

## **Ubuntu**

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform
terraform -v
```

---

# **2. Install & Configure AWS CLI (Windows + Ubuntu)**

## **Windows**

```powershell
choco install awscli -y
```

Verify it works:

```powershell
aws --version
```

---

## **Ubuntu**

```bash
sudo apt install awscli -y
aws --version
```

---

## **Configure AWS Credentials**

```bash
aws configure
```

Enter:

* AWS Access Key
* AWS Secret Key
* Region: `ap-south-1`
* Output: `json`

Credentials saved in:

```
~/.aws/credentials
```

---

# **3. Create Terraform Project Structure**

```
aws-terraform-demo/
│── main.tf
│── variables.tf
│── outputs.tf
```

---

# **4. Terraform Configuration (Provision EC2 Instance)**

### **main.tf**

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

resource "aws_instance" "my_vm" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "Terraform-EC2-Demo"
  }
}
```

---

### **variables.tf**

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

### **outputs.tf**

```hcl
output "public_ip" {
  value = aws_instance.my_vm.public_ip
}

output "instance_id" {
  value = aws_instance.my_vm.id
}
```

---

# **5. Run Terraform Commands (Windows PowerShell or Ubuntu)**

### **Initialize**

```bash
terraform init
```

### **Validate**

```bash
terraform validate
```

### **Plan**

```bash
terraform plan
```

### **Apply**

```bash
terraform apply
```

Confirm:

```
yes
```

### **Destroy**

```bash
terraform destroy
```

Confirm:

```
yes
```

---

# Thank you

> Keep Exploring, Keep Automating 🚀

---
