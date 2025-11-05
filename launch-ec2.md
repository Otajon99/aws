# 🚀 AWS EC2 Hands-On Lab

This lab walks you through launching, monitoring, securing, resizing, and managing an **Amazon EC2 instance** with **termination protection** and **stop protection**.

---

## 🧩 **Task 1: Launch Your Amazon EC2 Instance**

### Step 1: Access EC2
1. In the **AWS Management Console**, choose **Services → Compute → EC2**.  
2. Verify that your region is **N. Virginia (us-east-1)**.

---

### Step 2: Launch Instance
1. Choose **Launch instance**.
2. Name your instance **Web Server** (this creates a tag: `Name = Web Server`).

---

### Step 3: Select AMI
- Use the **default Amazon Linux 2023 AMI**.

> 🧠 An **AMI (Amazon Machine Image)** is a template that contains:
> - OS and applications
> - Launch permissions  
> - Block device mappings  

---

### Step 4: Choose Instance Type
- Keep **t2.micro** selected (1 vCPU, 1 GiB memory).

---

### Step 5: Key Pair (Login)
- Select existing key pair: **vockey**

> EC2 uses key pairs for secure login using public-key cryptography.

---

### Step 6: Configure Network Settings
1. Under **Network settings**, choose **Edit**.  
2. For **VPC**, select **Lab VPC**.  
3. Keep the **default subnet (PublicSubnet1)**.  
4. Create a new Security Group:

   | Field | Value |
   |--------|--------|
   | Security group name | Web Server security group |
   | Description | Security group for my web server |

5. Remove any default inbound rules.

---

### Step 7: Configure Storage
- Keep the **default 8 GiB** volume (EBS root volume).

---

### Step 8: Advanced Details
1. Expand **Advanced details**.  
2. Enable **Termination protection**.  
3. In **User data**, paste the script below:

   ```bash
   #!/bin/bash
   dnf install -y httpd
   systemctl enable httpd
   systemctl start httpd
   echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
