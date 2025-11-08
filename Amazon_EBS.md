# 🧪 Lab 4: Working with Amazon EBS

## 🏗️ Lab Overview

This lab focuses on **Amazon Elastic Block Store (Amazon EBS)** — a key storage service for Amazon EC2 instances.  
You’ll learn how to:
- Create an Amazon EBS volume
- Attach it to an instance
- Apply a file system
- Take a snapshot backup
- Restore from that snapshot

---

## 🧭 Topics Covered

By the end of this lab, you will be able to:

- ✅ Create an Amazon EBS volume  
- 🔗 Attach and mount your volume to an EC2 instance  
- 📸 Create a snapshot of your volume  
- 🪣 Create a new volume from your snapshot  
- 🧩 Attach and mount the new volume to your EC2 instance  

🕒 **Estimated Duration:** 30 minutes

---

## ⚠️ AWS Service Restrictions

In this lab environment, access to AWS services might be restricted to only those needed to complete the tasks.  
If you attempt to use other services, you may see permission errors.

---

## 💡 What is Amazon Elastic Block Store?

**Amazon Elastic Block Store (Amazon EBS)** provides **persistent block-level storage** for Amazon EC2 instances.

- 🧱 Volumes persist independently from the life of an instance.  
- 🔁 Automatically replicated within the same Availability Zone.  
- 💾 Can serve as boot partitions or attached as standard block devices.  
- ☁️ Snapshots stored in **Amazon S3**, replicated across multiple Availability Zones.  
- 🔐 Snapshots can be used for backup, cloning, or sharing.

For more info, see the [Amazon EBS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html).

---

## ⚙️ Amazon EBS Volume Features

| Feature | Description |
|----------|-------------|
| 💾 Persistent Storage | Volume lifetime independent of instance |
| 💻 General Purpose | Raw, unformatted block device |
| ⚡ High Performance | Equal or better than local EC2 drives |
| 🧩 High Reliability | Built-in redundancy within an AZ |
| 🔁 Designed for Resiliency | Annual Failure Rate: 0.1%–1% |
| 📏 Variable Size | From 1 GB to 16 TB |
| 🧠 Easy to Use | Create, attach, backup, restore, delete |

---

## 🔐 Accessing the AWS Management Console

1. Choose **Start Lab** at the top of the instructions.  
2. Wait until the timer appears and the **AWS link icon** turns green.  
3. Click the **AWS** link in the upper-left corner to open the console.  
4. Allow pop-ups if your browser blocks them.  
5. Arrange the console and lab instructions side by side for convenience.

---

## 🧾 Getting Credit for Your Work

At the end of the lab:
- Choose **Submit**
- Confirm with **Yes**
- Wait for the grades panel to appear  
- You can resubmit as many times as needed

> 💡 Tip: Names and configurations must be entered *exactly* as shown — they are case-sensitive.

---

## 🧱 Task 1: Create a New EBS Volume

1. In the **AWS Management Console**, search for and open **EC2**.  
2. Choose **Instances** — a pre-launched instance named `Lab` is available.  
3. Note its **Availability Zone** (e.g., `us-east-1a`).  
4. Go to **Volumes** → choose **Create volume**.  
5. Configure as follows:  
   - **Volume Type:** General Purpose SSD (gp2)  
   - **Size:** 1 GiB  
   - **Availability Zone:** Same as your instance  
6. Choose **Add tag**:  
   - **Key:** Name  
   - **Value:** My Volume  
7. Click **Create Volume**.  
8. Wait for it to move from **Creating → Available**.

---

## 🔗 Task 2: Attach the Volume to an Instance

1. Select **My Volume**.  
2. Choose **Actions → Attach volume**.  
3. In the **Instance** field, select **Lab**.  
4. The **Device name** will be `/dev/sdf`.  
5. Choose **Attach volume**.  
6. Confirm the state is now **In-use**.

---

## 💻 Task 3: Connect to Your Amazon EC2 Instance

1. In the **EC2 Console**, go to **Instances**.  
2. Select the **Lab** instance → choose **Connect**.  
3. On the **EC2 Instance Connect** tab → click **Connect**.  
4. A terminal opens with a `$` prompt.

---

## 🧩 Task 4: Create and Configure Your File System

1. View current storage:
   ```bash
   df -h
