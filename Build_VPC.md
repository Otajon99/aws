# 🧱 Build Your VPC and Launch a Web Server

## 📘 Overview
In this lab, you will use **Amazon Virtual Private Cloud (VPC)** to create a custom virtual network and deploy an **EC2 web server** inside it.

You will:
- Create a VPC
- Create public and private subnets
- Configure route tables, an Internet Gateway, and a NAT Gateway
- Create a Security Group
- Launch and configure an EC2 instance with a web server

---

## 🎯 Objectives
After completing this lab, you will be able to:

- Create a VPC and subnets  
- Configure a Security Group  
- Launch an EC2 instance into a subnet  
- Deploy and access a web server via HTTP

---

![Scenario](./vpc_structure.png)

## 🧩 Task 1: Create Your VPC

### Steps:
1. Open the **VPC Console** → choose **Create VPC** → select **VPC and more**
2. Configure the following:

| Setting | Value |
|----------|--------|
| Name tag | `lab-vpc` |
| IPv4 CIDR block | `10.0.0.0/16` |
| Number of AZs | `1` |
| Public subnet CIDR | `10.0.0.0/24` |
| Private subnet CIDR | `10.0.1.0/24` |
| NAT Gateway | In 1 AZ |
| DNS hostnames/resolution | Enabled |

3. Review and click **Create VPC**

### Components Created:
- **VPC:** `lab-vpc`
- **Public Subnet:** `lab-subnet-public1-us-east-1a (10.0.0.0/24)`
- **Private Subnet:** `lab-subnet-private1-us-east-1a (10.0.1.0/24)`
- **Internet Gateway:** `lab-igw`
- **NAT Gateway:** `lab-nat-public1-us-east-1a`
- **Route Tables:** `lab-rtb-public`, `lab-rtb-private1-us-east-1a`

---

## 🌍 Task 2: Create Additional Subnets

1. Go to **Subnets** → **Create subnet**
2. Create two new subnets:

| Name | AZ | CIDR Block |
|------|----|-------------|
| `lab-subnet-public2` | `us-east-1b` | `10.0.2.0/24` |
| `lab-subnet-private2` | `us-east-1b` | `10.0.3.0/24` |

3. Go to **Route Tables** and update associations:
   - For `lab-rtb-private1-us-east-1a`: Associate both private subnets  
   - For `lab-rtb-public`: Associate both public subnets

✅ **Result:**  
Your VPC now spans **two Availability Zones**, each with one **public** and one **private** subnet.

---

## 🛡️ Task 3: Create a Security Group

1. Navigate to **Security Groups** → **Create security group**
2. Configure the following:

| Setting | Value |
|----------|--------|
| Name | `Web Security Group` |
| Description | `Enable HTTP access` |
| VPC | `lab-vpc` |

### Add Inbound Rule:
| Type | Protocol | Port Range | Source | Description |
|------|-----------|-------------|---------|--------------|
| HTTP | TCP | 80 | 0.0.0.0/0 | Permit web requests |

Then click **Create Security Group**

---

## 🖥️ Task 4: Launch a Web Server Instance

1. Go to **EC2 Console** → **Launch instance**
2. Configure:

| Setting | Value |
|----------|--------|
| Name | `Web Server 1` |
| AMI | Amazon Linux 2023 |
| Instance Type | `t2.micro` |
| Key Pair | `vockey` |
| Network | `lab-vpc` |
| Subnet | `lab-subnet-public2` |
| Auto-assign public IP | Enabled |
| Security Group | `Web Security Group` |

3. Under **Advanced details → User data**, paste:

```bash
#!/bin/bash
# Install Apache Web Server and PHP
dnf install -y httpd wget php mariadb105-server
# Download Lab files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-ACCLFO-2/2-lab2-vpc/s3/lab-app.zip
unzip lab-app.zip -d /var/www/html/
# Turn on web server
chkconfig httpd on
service httpd start

## 🪜 Step 8: Launch and Connect to Your Instance

At the bottom of the **Summary** panel on the right side of the screen, choose **Launch instance**.

You will see a **Success** message.

---

### 🔍 View Your Instances

- Choose **View all instances**.
- Wait until **Web Server 1** shows **2/2 checks passed** in the **Status check** column.  
  > ⏳ This may take a few minutes.  
  You can choose the **Refresh** 🔄 icon at the top of the page every 30 seconds to check the latest status.

---

### 🌐 Connect to the Web Server

Once the instance is running:

1. Select **Web Server 1**.
2. Copy the **Public IPv4 DNS** value shown in the **Details** tab at the bottom of the page.
3. Open a new browser tab, paste the copied **Public DNS**, and press **Enter**.

You should now see a **web page displaying the AWS logo and instance meta-data values.**

---

### 🧱 Complete Architecture Deployed

You have successfully deployed the following architecture:

- **Amazon EC2 instance** running a web server
- **Security Group** allowing inbound HTTP (port 80) traffic
- **Elastic Block Store (EBS)** volume attached as root storage
- **Instance metadata** accessible via browser
- **Public IPv4 DNS** for external web access

🎉 **Congratulations!** You have successfully launched and connected to your first web server on Amazon EC2.
