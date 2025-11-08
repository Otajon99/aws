# ⚙️ Lab 6: Scale and Load Balance Your Architecture

## 📘 Lab Overview and Objectives
This lab walks you through using **Elastic Load Balancing (ELB)** and **Auto Scaling** services to distribute traffic and automatically scale your AWS infrastructure.

### 🧩 Key Services
- **Elastic Load Balancing (ELB):**  
  Automatically distributes incoming application traffic across multiple Amazon EC2 instances, improving fault tolerance and performance.
  
- **Auto Scaling:**  
  Automatically adjusts the number of EC2 instances based on demand. It ensures availability during traffic spikes and reduces costs during low usage.

### 🎯 Objectives
By the end of this lab, you will:
- Create an **Amazon Machine Image (AMI)** from a running instance  
- Create a **Load Balancer**  
- Create a **Launch Template** and **Auto Scaling Group**  
- Automatically scale instances based on demand  
- Create **Amazon CloudWatch alarms** and monitor infrastructure performance  

---

## ⏱ Duration
**Approximate time:** 30 minutes

---

## ⚙️ AWS Service Restrictions
In this lab, only required AWS services are available.  
Accessing unauthorized services may result in errors.

> ⚠️ **Caution:**  
> Attempting to launch **20 or more instances** will cause your AWS account to be **immediately deactivated** and all resources deleted.

---

## 🧩 Scenario

### Starting Architecture:
*(Initial setup before beginning the lab)*

### Final Architecture:
*(Desired state after completing the lab)*

---

## 🔐 Accessing the AWS Management Console
1. At the top of these instructions, choose **Start Lab**.  
   Wait until the **green circle** icon appears next to the AWS link.
2. Choose the **AWS** link to open the console in a new browser tab.  
   If blocked, allow pop-ups from your browser.
3. Arrange both tabs (lab instructions and console) side by side.

---

## 🧾 Getting Credit for Your Work
At the end of the lab, you will **submit your work** for scoring.

> 💡 **Tip:**  
> Enter all names and configurations **exactly** as specified (case-sensitive) to receive full credit.

---

## 🧱 Task 1: Create an AMI for Auto Scaling
You’ll create an **Amazon Machine Image (AMI)** from **Web Server 1** so that new instances can be launched with the same configuration.

1. In the AWS Console, search for and select **EC2**.  
2. In the left pane, choose **Instances**.  
3. Wait until **Web Server 1** shows `2/2 checks passed`.  
   - Click **Refresh** if necessary.  
4. Select **Web Server 1** → **Actions > Image and templates > Create image**.  
5. Configure:
   - **Image name:** `WebServerAMI`  
   - **Description:** `Lab AMI for Web Server`  
6. Choose **Create image**.  
   - A confirmation banner displays the **AMI ID**.

---

## 🌐 Task 2: Create a Load Balancer

### Step 1: Create a Target Group
1. In the left navigation pane, choose **Target Groups**.  
2. Choose **Create target group** and configure:
   - **Target type:** Instances  
   - **Target group name:** `LabGroup`  
   - **VPC:** `Lab VPC`  
3. Choose **Next** → Skip registering targets (none yet).  
4. Choose **Create target group**.

### Step 2: Create an Application Load Balancer
1. In the left pane, choose **Load Balancers** → **Create load balancer**.  
2. Under **Application Load Balancer**, choose **Create**.  
3. Configure:
   - **Load balancer name:** `LabELB`
4. Under **Network mapping:**
   - **VPC:** `Lab VPC`
   - Select:
     - `us-east-1a → Public Subnet 1`
     - `us-east-1b → Public Subnet 2`
5. Under **Security groups:**
   - Select **Web Security Group**
   - Remove the **default** security group  
6. Under **Listener (HTTP:80):**
   - Default action → **Forward to LabGroup**  
7. Choose **Create load balancer** → then **View load balancer**.  
   - Continue to next step (no need to wait for provisioning to finish).

---

## 🚀 Task 3: Create a Launch Template and Auto Scaling Group

### Step 1: Create a Launch Template
1. In the left pane, choose **Launch Templates** → **Create launch template**.  
2. Configure:
   - **Name:** `LabConfig`  
   - Enable: “Provide guidance for EC2 Auto Scaling”
   - **AMI:** `WebServerAMI`  
   - **Instance type:** `t2.micro`  
   - **Key pair:** `vockey`  
   - **Security groups:** `Web Security Group`
3. Expand **Advanced details** → enable **Detailed CloudWatch monitoring**.  
4. Choose **Create launch template**.

### Step 2: Create an Auto Scaling Group
1. From the success dialog, select **LabConfig** → **Actions > Create Auto Scaling group**.  
2. Step 1:  
   - **Name:** `Lab Auto Scaling Group`  
   - **Launch template:** `LabConfig`
3. Step 2:  
   - **VPC:** `Lab VPC`  
   - **Subnets:** `Private Subnet 1` and `Private Subnet 2`
4. Step 3:  
   - **Attach to existing load balancer**  
   - **Target group:** `LabGroup`  
   - Enable **Group metrics collection within CloudWatch**
5. Step 4: Configure scaling:
   | Setting | Value |
   |----------|--------|
   | Desired capacity | 2 |
   | Minimum capacity | 2 |
   | Maximum capacity | 6 |
   | Scaling policy name | `LabScalingPolicy` |
   | Metric type | Average CPU Utilization |
   | Target value | 60 |
6. Step 5: Skip notifications.  
7. Step 6: Add tag:
   - **Key:** `Name`  
   - **Value:** `Lab Instance`  
8. Choose **Create Auto Scaling group**.  
   - Initially shows 0 instances, then launches 2 automatically.

---

## 🔁 Task 4: Verify Load Balancing

1. In the left pane, choose **Instances** → confirm two **Lab Instance** entries.  
2. In **Target Groups** → select **LabGroup** → **Targets tab**.  
   - Wait until both instances show **Healthy**.
3. In **Load Balancers** → select **LabELB**.  
   - Copy the **DNS name** (omit “A Record”).  
   - Example:  
     ```
     LabELB-1998580470.us-west-2.elb.amazonaws.com
     ```
4. Open a browser → paste DNS name → press **Enter**.  
   - The application should appear — confirming successful load balancing.

---

## 📈 Task 5: Test Auto Scaling

### Step 1: Monitor Alarms
1. In the console, search for **CloudWatch** → choose **All alarms**.  
   - Two alarms should appear automatically (AlarmHigh & AlarmLow).  
2. If not, wait 60 seconds or refresh.

### Step 2: Adjust Scaling Threshold
If alarms are missing:
1. Go to **EC2 → Auto Scaling Groups → Lab Auto Scaling Group**.  
2. Under **Automatic Scaling tab**, select **LabScalingPolicy → Edit**.  
3. Change **Target value** to `50` → **Update**.  
4. Return to **CloudWatch → All alarms**.

### Step 3: Generate Load
1. In CloudWatch, confirm `AlarmHigh` shows **OK** (CPU < 60%).  
2. Return to the web app and choose **Load Test** beside the AWS logo.  
   - This generates high CPU usage across all instances.
3. In 3–5 minutes:
   - `AlarmHigh` should change to **In Alarm**.
   - Auto Scaling launches additional instances automatically.
4. Return to **EC2 → Instances** → confirm more than two **Lab Instance** entries.

---

## 🧹 Task 6: Terminate Web Server 1
Since Auto Scaling now handles your instances, terminate the original web server.

1. In **EC2 → Instances**, select **Web Server 1**.  
2. From **Instance state**, choose **Terminate instance** → confirm.

---

## 🧩 Summary
You have successfully:
- Created an **AMI** for scalable deployments  
- Built an **Application Load Balancer (ALB)**  
- Created a **Launch Template** and **Auto Scaling Group**  
- Configured **CloudWatch alarms** to trigger scaling actions  
- Verified load balancing and automatic scaling in action  

Your infrastructure now dynamically scales to meet demand while maintaining high availability and performance.

---
