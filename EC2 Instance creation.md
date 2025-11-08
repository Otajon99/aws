
## ☁️ Setting Up Your Secure VPC Environment

This exercise involves setting up a classic two-tier network architecture: a **public subnet** for your web server (accessible from the internet) and a **private subnet** (for future database or application servers) where external access is restricted.

### **Step 1: Create the Virtual Private Cloud (VPC)**

1. **Action:** Create a **VPC** using the default settings, specifying a suitable IP range (CIDR block), such as `10.0.0.0/16`.

2. **Why this is important:** The VPC is your **isolated, private virtual network** in the cloud. It's the secure container for all your resources. The `/16` range gives you a large pool of IP addresses (`10.0.0.1` to `10.0.255.254`) to use for your subnets and instances.

3. **Crucial Detail:** Ensure that both **DNS Hostnames** and **DNS Resolution** options are selected/enabled.

    - **Why:** This allows your instances to resolve public DNS names (like `google.com`) and automatically assigns a public DNS name to instances launched into a public subnet, which is essential for accessing your web server later.

---

### **Step 2: Create Subnets**

1. **Action:** Within your new VPC, create two distinct subnets:

    - **Public Subnet:** Give it a CIDR block like `10.0.1.0/24`. Choose an **Availability Zone (AZ)**.

    - **Private Subnet:** Give it a CIDR block like `10.0.2.0/24`. Choose a **different AZ** for high availability (best practice).

2. **Why this is important:**

    - **Subnets** are subdivisions of your VPC's IP range, dedicated to different security and accessibility needs. The `/24` block provides 256 IPs (251 usable) for each subnet.

    - **Availability Zones** are physically separate data centers. Placing resources in different AZs ensures that if one data center fails, your application remains available.

3. **Crucial Detail:** For the **Public Subnet**, modify its settings to **Enable auto-assign public IPv4 address**. This ensures that EC2 instances launched into this subnet automatically receive a public IP address, necessary for direct internet access.

---

### **Step 3: Create and Attach the Internet Gateway (IGW)**

1. **Action:** Create a new **Internet Gateway (IGW)**. Once created, **attach it** to your VPC.

2. **Why this is important:** The **IGW** is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet. It is the _connection point_ to the internet. **Note:** Simply attaching the IGW doesn't make a subnet public; that's the job of the routing table in the next step.

---

### **Step 4: Configure Routing Tables**

This is where you define the _traffic rules_ for your subnets. Every subnet **must** be associated with a routing table.

#### **A. Public Subnet Routing Table (The "Default" Table)**

1. **Action (Implicit/Check):** Your VPC creation automatically created a **Main Routing Table**. We will use this as the **Public Routing Table**.

2. **Action (Explicit):** Add the following route to this table (This addresses your original point #5):

    - **Destination:** `0.0.0.0/0`

    - **Target:** The **Internet Gateway (IGW)** you created in Step 3.

3. **Why this is important:**

    - `0.0.0.0/0` is the "catch-all" route—it means **"all traffic destined for the internet"**.

    - By pointing this traffic to the **IGW**, you are declaring that any subnet associated with this routing table is a **Public Subnet**.

4. **Action:** Associate your **Public Subnet** (`10.0.1.0/24`) with this routing table.

#### **B. Private Subnet Routing Table (Your Point #4)**

1. **Action:** Create a **new, separate routing table**. Name it something clear, like `Private-Route-Table`.

2. **Action:** Add **only** the default **Local Route** (`10.0.0.0/16` -> `local`). **Do not** add the `0.0.0.0/0` route pointing to the IGW.

3. **Why this is important:** By _not_ giving it a route to the IGW, you ensure that instances in this subnet cannot directly send or receive traffic from the internet, making it a true **Private Subnet**—perfect for databases.

4. **Action:** Associate your **Private Subnet** (`10.0.2.0/24`) with this new `Private-Route-Table`.

|**Route Table Component**|**Public Subnet Route Table**|**Private Subnet Route Table**|
|---|---|---|
|**Local Traffic**|`10.0.0.0/16` $\rightarrow$ `local`|`10.0.0.0/16` $\rightarrow$ `local`|
|**Internet Traffic**|`0.0.0.0/0` $\rightarrow$ **IGW**|**N/A** (No internet access)|

---

## 🛠️ Configuring and Launching the Web Server

### **Step 5: Create the Security Group (The EC2 Firewall)**

1. **Action:** Create a new **Security Group (SG)** within your VPC. Name it `Web-Server-SG`.

2. **Why this is important:** A Security Group acts as a **virtual firewall** for your EC2 instance, controlling traffic at the instance level. It is **stateful**, meaning you only define the inbound rules, and the response traffic is automatically allowed.

3. **Inbound Rules (What traffic is _allowed in_):**

    - **Rule 1 (HTTP):** Type: `HTTP`, Protocol: `TCP`, Port Range: `80`, Source: `0.0.0.0/0` (Allows _all_ internet users to view your website).

    - **Rule 2 (SSH):** Type: `SSH`, Protocol: `TCP`, Port Range: `22`, Source: `Your IP` (Allows _only you_ to securely connect to the instance's command line—replace `Your IP` with your current public IP address for security).

4. **Outbound Rules (What traffic is _allowed out_):**

    - **Action:** Modify the default outbound rule.

    - **Rule:** Type: `All Traffic`, Protocol: `All`, Port Range: `All`, Destination: `0.0.0.0/0` (Allows your server to send traffic anywhere). **Crucial Note (Your observation):** While the stateful nature of the SG often handles return traffic, allowing _all_ outbound traffic is a simple, robust way to ensure your web server can successfully download updates, install packages (like `httpd`), and reach external services. The **HTTPS** protocol (port 443) is included in "All Traffic." If you were to restrict the outbound rule, you would specifically need to allow ports 80 and 443 for web server updates and package installations.

---

### **Step 6: Launch EC2 Instance (Web Server)**

1. **Action:** Launch an **EC2 instance** (e.g., using Amazon Linux 2 AMI).

2. **Crucial Details:**

    - **VPC:** Select the VPC you created.

    - **Subnet:** Select the **Public Subnet** (`10.0.1.0/24`).

    - **Auto-assign Public IP:** Ensure this is **Enabled** (it should be if you completed Step 2 correctly).

    - **Security Group:** Select the `Web-Server-SG` you just created.

    - **Key Pair:** Select or create a Key Pair for SSH access.

---

### **Steps 7-10: Configure and Start the Web Server**

Once the EC2 instance is running, connect to it via SSH using the Key Pair you specified. Then, run these commands:

|**Step**|**Action**|**Command/Concept**|**Why this is important**|
|---|---|---|---|
|**7**|**Install Web Server**|`sudo yum update -y && sudo yum install httpd -y`|`yum` is the package manager. `httpd` is the Apache HTTP Web Server software.|
|**8**|**Create Web Page**|`sudo echo "<h1>Hello World from my VPC!</h1>" > /var/www/html/index.html`|This creates the default file the web server will display when someone visits the public IP address.|
|**9**|**Start Service**|`sudo systemctl start httpd`|This initiates the web server process, making it listen for requests on port 80.|
|**10**|**Enable at Boot**|`sudo systemctl enable httpd`|This ensures the web server automatically starts every time the instance is rebooted.|

---

### **Step 11: Access Your Web Server**

1. **Action:** Find the **Public IPv4 address** or **Public DNS** name of your running EC2 instance.

2. **Crucial Detail :** When you paste the DNS name into your browser, ensure you **remove any `https://`** that might be automatically copied from the AWS console and use `http://` instead, or simply the address.

    - **Why:** Your current security group is only configured to allow **HTTP (Port 80)** traffic. Your server is not configured to handle **HTTPS (Port 443)** yet (that requires a certificate), so the browser will time out if you try to use `https://`.

Your simple web page should now be visible on the internet!
