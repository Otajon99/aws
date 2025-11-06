## 🚦 AWS Network Security: Precedence & Flow

First, it's essential to understand the order in which your traffic is evaluated.

For traffic **coming into** your instance, the order is:

1. **Route Table** (Directs traffic to the subnet)
    
2. **Network ACL** (Filters traffic at the subnet boundary)
    
3. **Security Group** (Filters traffic at the instance)
    

For traffic **going out** of your instance, the order is reversed:

1. **Security Group** (Filters traffic leaving the instance)
    
2. **Network ACL** (Filters traffic leaving the subnet)
    
3. **Route Table** (Directs traffic to its next hop)
    

---

## 🗺️ Component Comparison Table

This table breaks down the key features of each component.

| **Feature**              | **🛣️ Route Table**                                                                        | **🛂 Network ACL (NACL)**                                                                                                             | **🛡️ Security Group (SG)**                                                                                             |
| ------------------------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Purpose**              | **Traffic Director**                                                                       | **Firewall (Subnet Level)**                                                                                                           | **Firewall (Instance Level)**                                                                                           |
| **Scope**                | Associated with a **Subnet**. (Controls where traffic _goes_ from the subnet).             | Associated with a **Subnet**. (Controls what traffic can _enter or leave_ the subnet).                                                | Associated with an **Instance (ENI)**. (Controls what traffic can _reach or leave_ the instance).                       |
| **State**                | N/A (Handles routing)                                                                      | **Stateless**                                                                                                                         | **Stateful**                                                                                                            |
| **Stateless / Stateful** | -                                                                                          | **Stateless:** Return traffic must be **explicitly allowed** by a corresponding rule (e.g., an outbound rule for an inbound request). | **Stateful:** Return traffic for an allowed inbound request is **automatically allowed**, regardless of outbound rules. |
| **Rule Type**            | Routes (Destination & Target)                                                              | **Allow** and **Deny** rules.                                                                                                         | **Allow** rules only. (Cannot create "deny" rules).                                                                     |
| **Rule Evaluation**      | **Longest Prefix Match:** The most specific route that matches the destination IP is used. | **Numbered List:** Rules are evaluated in order, from the lowest rule number to the highest. The first matching rule is applied.      | **All Rules:** All "allow" rules are evaluated. If any rule matches, the traffic is permitted.                          |

---

## 🏗️ Architecting Your VPC with Route Tables

You asked if you need a separate route table for each subnet. The answer is: **No, you don't _have_ to, but you absolutely _should_ as a best practice.**

The route table is what defines a subnet as "public" or "private."

- A **Public Subnet** has a route table with a route to an Internet Gateway (IGW).
    
- A **Private Subnet** does not have a route to an IGW.
    

Relying on the single "Main Route Table" is a security risk. If you add an internet route to it, _all_ your subnets, including your databases, become public.

### How to Set Up VPC Routing (Best Practice)

1. **The Main Route Table (Your "Isolated" Default)**
    
    - **Action:** When your VPC is created, find the **Main Route Table**.
        
    - **Configuration:** **Do not add any routes to it.** Leave it with only the default `local` route (e.g., `10.0.0.0/16 -> local`).
        
    - **Result:** This table has no path to the internet. Any subnet left here is **isolated**. This is perfect for your most secure resources, like databases.
        
2. **Create a "Public" Route Table**
    
    - **Action:** Create a new **Custom Route Table** (e.g., `public-rt`).
        
    - **Configuration:** Add a route for `0.0.0.0/0` (all traffic) with the target set to your `Internet Gateway (igw-...)`.
        
    - **Association:** Explicitly associate your **public subnets** (for web servers, load balancers) with this table.
        
3. **Create a "Private-NAT" Route Table (Optional)**
    
    - **Action:** Create another **Custom Route Table** (e.g., `private-nat-rt`).
        
    - **Configuration:** Add a route for `0.0.0.0/0` with the target set to your `NAT Gateway (nat-...)`. (The NAT Gateway itself must live in one of your public subnets).
        
    - **Association:** Associate your **private application subnets** (for app servers that need to download updates) with this table.
        

This setup gives you three distinct, secure routing profiles:

|**Route Table Name**|**Associated Subnet(s)**|**Key Route (Destination → Target)**|**Result**|
|---|---|---|---|
|**Main Route Table**|Private DB Subnets|`10.0.0.0/16` $\rightarrow$ `local` (Only)|**Isolated.** No internet access at all.|
|**Custom Public RT**|Public Web Subnets|`0.0.0.0/0` $\rightarrow$ `Internet Gateway`|**Public.** Two-way internet access.|
|**Custom Private RT**|Private App Subnets|`0.0.0.0/0` $\rightarrow$ `NAT Gateway`|**Private.** Outbound-only internet access.|

---

## 🛡️ Recommended Security Strategy: Defense in Depth

Now that your network architecture is solid, you apply layered security. **Do not leave NACLs wide open.** Use both SGs and NACLs for a "defense in depth" strategy.

- **Security Group (SG):** This is your instance's **personal bodyguard**. It's stateful, smart, and only lets in exactly who is on the guest list.
    
- **Network ACL (NACL):** This is the **security gate for the entire neighborhood** (your subnet). It's a broad, stateless guard that blocks obvious threats.
    

### 1. Security Groups (Your Primary, Strict Firewall)

This is where you should be **most strict**. SGs are stateful and your main line of defense.

- **Principle:** Least Privilege.
    
- **How to use:** Be as specific as possible. Instead of allowing port 3306 (MySQL) from an _IP range_, allow it _only_ from the **Security Group** of your web servers (e.g., source: `sg-abcdef123`).
    

### 2. Network ACLs (Your Secondary, Broad Firewall)

This is your safety net. Use custom NACLs to provide a broader layer of security.

- **Principle:** Subnet-level protection and blocking known threats.
    
- **How to use:**
    
    - **Block known bad IPs:** If you have a list of malicious IPs, you can add an explicit `Deny` rule. SGs cannot do this.
        
    - **Enforce subnet-wide rules:** For your `main-rt` database subnet, you can add a `Deny` rule to the NACL for all outbound traffic to `0.0.0.0/0`. This acts as a backup in case a Security Group is misconfigured.
        
    - **Remember Statelessness:** If you allow inbound port 443, you must _also_ allow outbound ephemeral ports (`1024-65535`) for the return traffic.
        

---

## 💡 Putting It All Together: 2-Tier Example

Here is how all three components work together in a standard public/private setup.

|**Tier / Subnet**|**🛣️ Route Table**|**🛂 Network ACL (NACL) Rules**|**🛡️ Security Group (SG) Rules**|
|---|---|---|---|
|**Public Web Subnet**|`public-rt` (Routes `0.0.0.0/0` to **IGW**)|**Inbound:** `Allow` TCP 443, `Allow` Ephemeral (1024-65535).<br><br>  <br><br>**Outbound:** `Allow` TCP 443, `Allow` Ephemeral.|**Web-Server-SG:**<br><br>  <br><br>**Inbound:** `Allow` TCP 443 from `0.0.0.0/0`.<br><br>  <br><br>**Outbound:** `Allow all` (for updates, etc.).|
|**Private DB Subnet**|`main-rt` (No route to `0.0.0.0/0`)|**Inbound:** `Allow` TCP 3306 from [Web Subnet CIDR], `Allow` Ephemeral.<br><br>  <br><br>**Outbound:** `Allow` TCP 3306, `Allow` Ephemeral.|**DB-Server-SG:**<br><br>  <br><br>**Inbound:** `Allow` TCP 3306 _only from_ `Web-Server-SG`.<br><br>  <br><br>**Outbound:** `Allow all` (can be locked down).|
