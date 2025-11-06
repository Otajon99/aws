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

---

## 🧭 Step 9: Launch the Instance

1. At the bottom of the **Summary** panel, choose **Launch instance**.  
   You will see a **Success** message.

2. Choose **View all instances**.

3. In the **Instances list**, select **Web Server**.

4. Review details such as:
   - **Instance type**
   - **Security settings**
   - **Network settings**

   The instance is assigned a **Public IPv4 DNS** that you can use to contact it from the Internet.

---

### 🕒 Instance Status Flow

| Status | Meaning |
|--------|----------|
| Pending | Instance is being launched |
| Initializing | System is setting up |
| Running | Instance is ready |

✅ Wait for:
Instance State: Running
Status Checks: 2/2 checks passed

markdown
Copy code

🎉 **Congratulations!** You have successfully launched your first Amazon EC2 instance.

---

## 📊 Task 2: Monitor Your Instance

Monitoring is important for maintaining reliability, availability, and performance.

### 1. Check Status
- Choose the **Status checks** tab.
- Confirm both:
  - **System reachability**
  - **Instance reachability**
  
  Both should display as **Passed** ✅.

### 2. CloudWatch Monitoring
- Choose the **Monitoring** tab.
- Amazon EC2 sends metrics to **CloudWatch**.
- Basic (5-minute) monitoring is enabled by default.
- You can enable **Detailed (1-minute)** monitoring.

To enlarge any metric graph → click the **three dots (⋮)** → **Enlarge**.

### 3. View System Log
Go to:
Actions → Monitor and troubleshoot → Get system log

markdown
Copy code

This displays the console output, useful for diagnosing kernel or configuration issues.

- You should see that the **HTTP package** was installed from your **user data script**.

### 4. Instance Screenshot
Actions → Monitor and troubleshoot → Get instance screenshot

markdown
Copy code

This shows the current console view of your instance.

🎉 You’ve now explored multiple EC2 monitoring options.

---

## 🔒 Task 3: Update Your Security Group and Access the Web Server

Your web server script is running — let’s access it.

1. Select **Web Server** → **Details tab**.
2. Copy the **Public IPv4 address**.
3. Open your browser → paste IP → press **Enter**.

❌ **Problem:** The site won’t load — HTTP (port 80) isn’t open yet.

### 🔧 Fix by Editing Security Group Rules

1. In the left navigation, choose **Security Groups**.
2. Select **Web Server security group**.
3. Choose **Inbound rules → Edit inbound rules**.
4. Add a rule:
Type: HTTP
Source: Anywhere-IPv4

markdown
Copy code
5. Choose **Save rules**.

Now, refresh your browser tab — you should see:

Hello From Your Web Server!

yaml
Copy code

🎉 **Success!** You modified your security group to allow web traffic.

---

## ⚙️ Task 4: Resize Your Instance and EBS Volume

You can adjust the instance type and storage when needs change.

### 1. Stop the Instance
EC2 Console → Instances → Web Server → Instance state → Stop instance

bash
Copy code

Wait until:
Instance State: Stopped

shell
Copy code

### 2. Change Instance Type
Actions → Instance settings → Change instance type

markdown
Copy code
- **New type:** `t2.small`  
- Choose **Apply**

### 3. Enable Stop Protection
Actions → Instance settings → Change stop protection → Enable → Save

yaml
Copy code

🧠 Note: Stopping doesn’t delete the instance — data and EBS volumes are retained.

### 4. Resize the EBS Volume
Storage tab → Select Volume ID → Actions → Modify volume

markdown
Copy code
- **Change size:** 10 GiB  
- Choose **Modify** → **Confirm**

### 5. Start the Instance
Instance state → Start instance

yaml
Copy code

🎉 Your EC2 instance is now upgraded to:
- **Instance Type:** t2.small  
- **Disk Size:** 10 GiB

---

## 📈 Task 5: Explore EC2 Limits

Each AWS account has regional limits for EC2 resources.

1. Open the AWS Console → Search for **Service Quotas**.
2. Choose **AWS services → Amazon EC2**.
3. In **Find quotas**, search:  
   `running on-demand`

Observe limits like:
- Number of on-demand instances
- Type-specific quotas

🧠 You can **request limit increases** if needed.

---

## 🧪 Task 6: Test Stop Protection

Stop protection prevents accidental instance stops.

### 1. Try Stopping
Instance state → Stop instance

yaml
Copy code
You’ll see:
Failed to stop the instance. Modify 'disableApiStop' to try again.

shell
Copy code

### 2. Disable Stop Protection
Actions → Instance settings → Change stop protection → Uncheck Enable → Save

yaml
Copy code

Now stop the instance again.

🎉 **Success!** Stop protection worked — and you learned how to safely stop your instance.

---

## ✅ Summary

You have:
- Launched and monitored an EC2 instance  
- Configured security groups  
- Resized both instance type and EBS volume  
- Explored service limits  
- Tested stop protection  

---

**🎊 Congratulations! You’ve completed your EC2 management lab.**
