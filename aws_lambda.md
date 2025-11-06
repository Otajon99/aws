# 🧩 Activity: AWS Lambda

## Lab Overview
In this hands-on activity, you will create an **AWS Lambda function** that stops an **Amazon EC2** instance every minute using **Amazon EventBridge**.  
The Lambda function uses an **IAM role** to gain permissions to stop the instance.

---

## 🕒 Duration
**Approximate time:** 30 minutes

---

## ⚠️ AWS Service Restrictions
In this lab environment, access to some AWS services may be limited to only those required for this activity.  
Errors may occur if you try to access or perform other actions beyond these instructions.

---

## 🔑 Accessing the AWS Management Console

1. At the top of the lab page, choose **Start Lab**.  
2. Wait for the **AWS** link in the upper-left corner to turn **green**.  
3. Choose the **AWS** link to open the AWS Management Console in a new browser tab.  
   - If the tab doesn’t open, allow pop-ups in your browser.  
4. Arrange your screen so you can view these instructions and the console side-by-side.

---

## 🧾 Getting Credit for Your Work
At the end of the lab, choose **Submit** to receive a score.  
Values shown **in this format** must be entered exactly (case-sensitive).

---

## 🧠 Task 1: Create a Lambda Function

1. In the AWS Console search bar, search for and open **Lambda**.
2. Choose **Create function**.
3. Configure the following:
   - **Author from scratch**
   - **Function name:** `myStopinator`
   - **Runtime:** `Python 3.11`
4. Under **Change default execution role**, select:
   - **Execution role:** `Use an existing role`
   - **Existing role:** Choose `myStopinatorRole` from the dropdown.
5. Choose **Create function**.

---

## ⚙️ Task 2: Configure the Trigger

You will create a scheduled event to trigger the Lambda function every minute using **EventBridge**.

1. Choose **Add trigger**.
2. From the **Select a trigger** dropdown, select **EventBridge (CloudWatch Events)**.
3. Configure:
   - **Rule name:** `everyMinute`
   - **Rule type:** `Schedule expression`
   - **Schedule expression:** `rate(1 minute)`
4. Choose **Add**.

---

## 🧩 Task 3: Configure the Lambda Function Code

1. In the **Function overview** panel, select **Code**, then choose **lambda_function.py**.
2. Delete the existing code and paste the following:

   ```python
   import boto3
   region = '<REPLACE_WITH_REGION>'
   instances = ['<REPLACE_WITH_INSTANCE_ID>']
   ec2 = boto3.client('ec2', region_name=region)

   def lambda_handler(event, context):
       ec2.stop_instances(InstanceIds=instances)
       print('stopped your instances: ' + str(instances))
Replace:

<REPLACE_WITH_REGION> with your AWS Region code (e.g., 'us-east-1').

<REPLACE_WITH_INSTANCE_ID> with your EC2 instance ID (e.g., 'i-0123456789abcdef0').

Choose File → Save, then Deploy.

✅ Your Lambda function is now configured to automatically stop your instance every minute.

📊 Task 4: Verify That the Lambda Function Worked
Go to the Amazon EC2 console.

Check the status of your instance named instance1.
It should soon show as stopped.

You can refresh using the 🔄 icon.

Try to start the instance again — it will stop automatically once the Lambda function triggers again.

🧾 Submitting Your Work
Choose Submit at the top of the lab page.

Choose Yes when prompted.

Wait a few minutes for grading to complete.

If your score doesn’t appear, wait at least 5 minutes and submit again.

To view detailed feedback, choose Submission Report.

🎉 Activity Complete
✅ Congratulations! You have successfully:

Created a Lambda function (myStopinator)

Configured a scheduled EventBridge trigger (everyMinute)

Automated EC2 instance shutdowns

Verified successful execution and monitoring

🏗️ Architecture Diagram
(Add diagram here if applicable — for GitHub, you can embed using:)

markdown
Copy code
![AWS Lambda Architecture](link-to-your-diagram.png)
Author: AWS Academy Hands-on Lab
Modified for GitHub Markdown by: O.B.
Date: 2025-11-05
