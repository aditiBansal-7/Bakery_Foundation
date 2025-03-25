# 🍞 Bakery Foundation Example on Windows

## Overview
This guide provides step-by-step instructions on setting up and using Packer to create a machine image (AMI) on AWS. It covers installation, configuration, and deployment on Windows.

## 📌 Prerequisites
Before starting, ensure you have:
- A Windows machine with administrator access.
- An AWS account with IAM credentials.
- Basic knowledge of AWS and PowerShell.

---

## ⚙️ Step 1: Install Required Tools

### 🔹 1.1 Install Packer

#### ✅ Download Packer
1. Open your browser and go to the [Packer Download Page](https://developer.hashicorp.com/packer/downloads).
2. Download the latest Windows (64-bit) ZIP file.

#### ✅ Extract Packer
1. Navigate to the downloaded ZIP file.
2. Right-click and select **Extract All...**
3. Move `packer.exe` to `C:\packer` (Create this folder if it doesn’t exist).

#### ✅ Add Packer to System PATH
1. Open **Environment Variables** (Search for it in Windows).
2. Click **Environment Variables** → Under **System Variables**, find **Path** → Click **Edit**.
3. Click **New**, then add:
   ```
   C:\packer
   ```
4. Click **OK** and close all windows.

#### ✅ Verify Packer Installation
Open PowerShell and run:
```powershell
packer --version
```
![WhatsApp Image 2025-03-25 at 21 00 56_b733a26a](https://github.com/user-attachments/assets/f63cf78e-1f1a-4158-987e-5647cea08135)

Expected Output:
```
1.9.2  # (or your installed version)
```

---

### 🔹 1.2 Install AWS CLI

#### ✅ Download & Install AWS CLI
1. Go to the [AWS CLI Download Page](https://aws.amazon.com/cli/).
2. Download and run the `AWSCLI.msi` installer.
3. Follow the on-screen steps: **Next** → **Next** → **Finish**.

#### ✅ Verify Installation
```powershell
aws --version
```
Expected Output:
```
aws-cli/2.x.x Windows/10
```

---

### 🔹 1.3 Configure AWS CLI (5 minutes)
Run the following command in PowerShell:
```powershell
aws configure
```
Enter the following when prompted:
```
AWS Access Key ID: <Your AWS Key>
AWS Secret Access Key: <Your AWS Secret>
Default region name: us-east-1 (or your preferred region)
Default output format: json (Press Enter)
```
✅ AWS CLI is now configured!

---
![WhatsApp Image 2025-03-25 at 21 06 03_7a416110](https://github.com/user-attachments/assets/1a502b73-8d7a-49e4-ae19-fad966c25de7)

## 🏗️ Step 2: Create the Packer Template

### 🔹 2.1 Create the Packer HCL File
1. Open **Notepad** or **VS Code**.
2. Copy the following code into a new file:
```hcl
packer {
  required_plugins {
    amazon = {
      source  = "github.com/hashicorp/amazon"
      version = ">= 1.0.0"
    }
  }
}

variable "aws_region" {
  default = "us-east-1"
}

source "amazon-ebs" "python39" {
  ami_name      = "bakery-foundation-python39-${formatdate("YYYYMMDD-HHmmss", timestamp())}"
  instance_type = "t2.micro"
  region        = var.aws_region
  source_ami    = "ami-0a25f237e97fa2b5e"
  ssh_username  = "ubuntu"
}

build {
  sources = ["source.amazon-ebs.python39"]

  provisioner "shell" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y python3.9 python3.9-venv python3.9-dev"
    ]
  }
}
```
3. Save the file as `bakery.pkr.hcl` in `C:\packer`.

#### ✅ Find a Valid Ubuntu AMI
Run the following AWS CLI command to get the latest Ubuntu AMI:
```powershell
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*" --query "Images | sort_by(@, &CreationDate)[-1].ImageId" --output text
```
✅ Update `bakery.pkr.hcl` by replacing `source_ami` with the new AMI ID.

---

## 🚀 Step 3: Validate and Build the Image

### 🔹 3.1 Initialize and Validate Packer Template
Open PowerShell and navigate to `C:\packer`:
```powershell
cd C:\packer
```
Initialize Packer:
```powershell
packer init .
```
Validate the template:
```powershell
packer validate bakery.pkr.hcl
```
✅ Expected Output: `The configuration is valid.`

### 🔹 3.2 Build the Machine Image
Run the following command:
```powershell
packer build bakery.pkr.hcl
```
![WhatsApp Image 2025-03-25 at 21 06 57_53013da6](https://github.com/user-attachments/assets/84f62208-5f17-4754-9cb4-08da0274cc35)

This will:
- Create a temporary EC2 instance.
- Install Python 3.9.
- Convert it into an Amazon Machine Image (AMI).
- Delete the temporary instance.

---

## 🎯 Step 4: Deploy and Test the AMI

### 🔹 4.1 Find the AMI
1. Log in to AWS Console.
2. Navigate to **EC2 → AMIs** (Set the region you used when creating the AMI).
3. Find the AMI named: `bakery-foundation-python39-timestamp`

   ![WhatsApp Image 2025-03-25 at 21 27 55_88dc11e9](https://github.com/user-attachments/assets/b0d8b4e1-f0a4-4414-9c87-7c3c84dbac9d)


### 🔹 4.2 Launch an EC2 Instance with Your AMI
1. Go to **AWS EC2 Console**: [AWS EC2 Dashboard](https://console.aws.amazon.com/ec2/).
2. Click **Launch Instance** → **My AMIs** (Left Sidebar).
3. Search for your AMI and **Select It**.
4. Choose:
   - **Instance Type**: `t2.micro` (or higher, based on your needs).
   - **Key Pair**: Use an existing key or create a new one.
   - **Security Group**: Allow SSH (port `22`) and other required ports.
5. Click **Launch!** 🚀
   ![WhatsApp Image 2025-03-25 at 21 28 43_9d1e23f7](https://github.com/user-attachments/assets/99314e82-a384-46d8-8bbd-f2a09083bcef)


### 🔹 4.3 Connect to the Instance
Get the **Public IP** from the EC2 Console.

Open PowerShell and connect via SSH:
```powershell
ssh -i "C:\path\to\your-key.pem" ubuntu@your-instance-ip
```
✅ Accept the SSH key fingerprint (First Time Only): Type `yes` and press Enter.

### 🔹 4.4 Verify Python Installation
Once inside the instance, run:
```powershell
python3.9 --version
```
✅ Expected Output:
```
Python 3.9.5
```

---

## 🎉 Conclusion
You have successfully:
✅ Set up Packer.
✅ Created an AWS AMI.
✅ Deployed an EC2 instance with Python 3.9.

🚀 Now you can use your instance for Python development on AWS!

