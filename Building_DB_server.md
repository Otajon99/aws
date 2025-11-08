# 🧪 Lab 5: Build Your DB Server and Interact With Your DB Using an App

## 📘 Lab Overview and Objectives
This lab demonstrates how to leverage an **AWS-managed database instance** to meet relational database needs using **Amazon RDS**.

**Amazon RDS** simplifies the process of setting up, operating, and scaling a relational database in the cloud. It handles database administration tasks while providing cost-efficient, resizable capacity so you can focus on applications and business logic.

RDS supports six popular database engines:
- Amazon Aurora  
- Oracle  
- Microsoft SQL Server  
- PostgreSQL  
- MySQL  
- MariaDB  

### 🎯 Objectives
By the end of this lab, you will be able to:
- Launch an **Amazon RDS DB instance** with high availability  
- Configure the DB instance to permit connections from a web server  
- Open a web application and interact with the database  

---

## ⏱ Duration
Approximate time: **30 minutes**

---

## ⚙️ AWS Service Restrictions
Your lab environment is limited to services required for completing this exercise.  
Attempting to use other services or actions may result in access errors.

---

## 🧩 Scenario
### Initial Infrastructure:
*(Provided at the start of the lab)*

### Final Infrastructure:
*(After completing the lab)*

---

## 🔐 Accessing the AWS Management Console
1. At the top of these instructions, choose **Start Lab**.  
   - Wait for the **green circle icon** next to the AWS link (top-left corner) before proceeding.
2. Click the **AWS** link to open the console in a new tab.  
   - If the tab doesn’t open, enable pop-ups in your browser.
3. Arrange both tabs (instructions and console) side-by-side for convenience.

---

## 🧾 Getting Credit for Your Work
At the end of the lab, you’ll **submit your work** to receive a score.

> 💡 **Tip:**  
> The grading script checks **names and configurations exactly** as specified (case-sensitive).  
> Enter values exactly as shown in **This Format**.

---

## 🧱 Task 1: Create a Security Group for the RDS DB Instance
You will create a **security group** that allows your **web server** to access your **RDS instance**.

1. In the AWS Console, search for and open **VPC**.
2. From the left pane, choose **Security Groups**.
3. Choose **Create security group** and configure:

   | Setting | Value |
   |----------|--------|
   | Security group name | `DB Security Group` |
   | Description | `Permit access from Web Security Group` |
   | VPC | `Lab VPC` |

4. Under **Inbound rules**, choose **Add rule**:
   - **Type:** MySQL/Aurora (3306)  
   - **Source:** Start typing `sg` and select **Web Security Group**  

5. Choose **Create security group**.  
   - You’ll use this when launching the RDS instance.

---

## 🌐 Task 2: Create a DB Subnet Group
You will now define **subnets** that RDS can use.

1. In the AWS Console, search for **RDS**.
2. In the left pane, choose **Subnet groups**.
3. Choose **Create DB Subnet Group** and configure:

   | Setting | Value |
   |----------|--------|
   | Name | `DB-Subnet-Group` |
   | Description | `DB Subnet Group` |
   | VPC | `Lab VPC` |

4. Under **Add subnets**, expand **Availability Zones** and select:
   - `us-east-1a`
   - `us-east-1b`

5. Expand **Subnets** and select:
   - `10.0.1.0/24`
   - `10.0.3.0/24`

6. Choose **Create**.

---

## 🗃️ Task 3: Create an Amazon RDS DB Instance
You will create a **Multi-AZ RDS MySQL** instance.

1. In the left pane, select **Databases** → **Create database**.
2. Under **Engine options**, choose **MySQL**.
3. Under **Templates**, select **Dev/Test**.
4. Under **Availability and durability**, choose **Multi-AZ DB instance**.

### Configure Database Settings:
| Setting | Value |
|----------|--------|
| DB instance identifier | `lab-db` |
| Master username | `main` |
| Master password | `lab-password` |
| Confirm password | `lab-password` |

### Instance Class:
- Select **Burstable classes (includes t classes)**
- Choose **db.t3.micro**

### Storage:
| Setting | Value |
|----------|--------|
| Storage type | General Purpose (SSD) |
| Allocated storage | 20 GB |

### Connectivity:
| Setting | Value |
|----------|--------|
| VPC | Lab VPC |
| Existing VPC security groups | `DB Security Group` |
| Deselect | `default` |

### Monitoring:
- Expand **Additional configuration**
- Uncheck **Enable Enhanced monitoring**

### Additional Configuration:
| Setting | Value |
|----------|--------|
| Initial database name | `lab` |
| Automatic backups | Disabled |
| Encryption | Disabled |

> ⚠️ **Note:** Disabling backups is only for this lab — not for production.

5. Choose **Create database**  
   - If you see a `not authorized to perform: iam:CreateRole` error, ensure **Enhanced monitoring** is unchecked.

6. Once created, select **lab-db** and wait until the **Status** is `Modifying` or `Available`.

7. Scroll to **Connectivity & security**, copy the **Endpoint** (e.g.,  
   `lab-db.xxxxx.us-east-1.rds.amazonaws.com`)  
   Save it — you’ll use it soon.

---

## 💻 Task 4: Interact With Your Database
You’ll now connect your web app to the new RDS instance.

1. Open the **AWS Details** dropdown (top of lab) and copy the **WebServer IP address**.
2. Open a new browser tab → Paste the IP → Press **Enter**.
3. The **web app** will load showing EC2 instance info.
4. Click the **RDS** link at the top.

### Configure Database Connection:
| Field | Value |
|--------|--------|
| Endpoint | *(Paste the endpoint from Task 3)* |
| Database | `lab` |
| Username | `main` |
| Password | `lab-password` |

5. Click **Submit**.

You’ll see a message confirming data transfer. Then, the **Address Book app** will appear.

### ✅ Test the App:
- Add, edit, and delete contacts.  
- Data is saved to the RDS DB and replicated to a standby instance.

---

## 🧩 Summary
You have successfully:
- Created an **RDS Multi-AZ MySQL** database  
- Configured **security and subnet groups**  
- Connected a **web app** to your database  
- Verified data persistence and replication  

---
