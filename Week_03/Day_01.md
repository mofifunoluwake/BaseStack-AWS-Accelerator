# ☁️ Week 3 · Day 1: VPC Fundamentals

**Module:** Networking & VPC  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

When you launch an EC2 instance, create an RDS database, or deploy any meaningful AWS application — every single one of those resources lives inside a network. That network is the **Amazon Virtual Private Cloud (VPC)**.

A VPC is your own logically isolated section of the AWS cloud. It looks, behaves, and is configured exactly like a traditional on-premise data centre network — but you manage it entirely through software, with no physical hardware to purchase, rack, or cable.

**Without understanding VPCs, you cannot reason about:**
- Network security
- High availability
- Internet access
- Private connectivity
- Cost architecture

**For Nigerian banks and fintechs:** CBN requirements demand network segmentation — production databases in private subnets with no direct internet exposure. Understanding VPC is the difference between compliant and non-compliant cloud architecture.

**The career truth:** The difference between a junior cloud engineer and a mid-level one is largely VPC. Build the mental model now.

---

## 2. The Core Building Blocks

### VPC (Virtual Private Cloud)

| What it is | Your own isolated, software-defined network inside an AWS Region |
|------------|----------------------------------------------------------------|
| What you control | IP address space, subnets, routing, firewall rules |
| Scope | One VPC spans **all Availability Zones** in a Region |
| Default behavior | Nothing enters or leaves your VPC unless you explicitly allow it |

**Analogy:** The fenced compound of your company's estate. Inside the fence, you control what moves where. Outside is the public internet.

### CIDR Block

| What it is | The method used to define your VPC's IP address range |
|------------|-------------------------------------------------------|
| Format | IP address + prefix length, e.g., `10.0.0.0/16` |
| What the prefix means | Number of bits that are FIXED (network portion). Remaining bits are variable (host addresses). |
| Critical rule | You choose your CIDR when creating the VPC and **cannot change it easily afterward** — plan carefully |

**CIDR Math (understand this for the exam):**

| CIDR | Variable bits | Total IPs | Usable IPs (AWS reserves 5) |
|------|---------------|-----------|----------------------------|
| `/16` | 16 bits | 65,536 | 65,531 |
| `/24` | 8 bits | 256 | 251 |
| `/28` | 4 bits | 16 | 11 |

**AWS reserves 5 IPs per subnet:** .0 (network address), .1 (VPC router), .2 (DNS), .3 (future use), .255 (broadcast)

### Subnet

| What it is | A subdivision of your VPC's IP space |
|------------|--------------------------------------|
| AZ binding | Tied to **exactly ONE Availability Zone** (subnets are zonal) |
| What you deploy into it | EC2 instances, RDS databases, Lambda ENIs |
| Public vs Private | Public subnet = has a route to an Internet Gateway. Private subnet = no direct internet route. |
| Critical rule | A subnet is public because of its **ROUTE TABLE** — not because of its name |

### Route Table

| What it is | A set of rules that determines where network traffic from a subnet is directed |
|------------|-------------------------------------------------------------------------------|
| Binding | Every subnet must be associated with exactly one route table |
| What routes look like | "Traffic to `0.0.0.0/0` (the internet) goes to the Internet Gateway" |
| Local route | Every route table automatically has a `local` route for VPC-internal traffic (`10.0.0.0/16 → local`) |

### Internet Gateway (IGW)

| What it is | A horizontally-scaled, redundant AWS-managed gateway |
|------------|------------------------------------------------------|
| What it does | Enables communication between your VPC and the public internet |
| What makes a subnet public | Its route table has a `0.0.0.0/0` route pointing to an IGW |
| Critical rule | Without an IGW route, a subnet is **private** regardless of any other setting |

### NAT Gateway

| What it is | Allows EC2 instances in **private subnets** to initiate **outbound** internet connections |
|------------|-----------------------------------------------------------------------------------------|
| What it does NOT do | Does NOT allow inbound connections from the internet |
| Where it lives | In a **public subnet** with an Elastic IP |
| How private instances use it | Private subnet route table has `0.0.0.0/0 → NAT Gateway` |
| Common use | Downloading software updates, calling external APIs from private instances |

---

## 3. Security Layers — Security Groups vs NACLs

This is the **most tested distinction** in VPC security on the SAA-C03 exam.

| Feature | Security Group | Network ACL (NACL) |
|---------|----------------|-------------------|
| **Level** | Instance-level (attached to EC2 ENI) | Subnet-level (applies to entire subnet) |
| **State** | **STATEFUL** — return traffic auto-allowed | **STATELESS** — must explicitly allow both directions |
| **Rules** | Allow rules only (no explicit deny) | Allow AND Deny rules |
| **Evaluation** | All rules evaluated — most permissive wins | Rules evaluated in number order — first match wins |
| **Default inbound** | DENY all (nothing allowed until you add rules) | ALLOW all (default NACL allows everything) |
| **Default outbound** | ALLOW all | ALLOW all |

**The critical difference to memorize:**

- **Security Group (stateful):** You allow inbound HTTP on port 80. The response traffic is **automatically allowed**. No outbound rule needed.
- **NACL (stateless):** You allow inbound HTTP on port 80. You must ALSO add an outbound rule for the **ephemeral port range (1024–65535)** or the response traffic will be blocked.

---

## 4. How It Works — Building a Production VPC Step by Step

### Step 1: Design Your IP Space First (Before Clicking Anything)

1. Choose your VPC CIDR block — a `/16` (e.g., `10.0.0.0/16`) gives you 65,536 IPs
2. Plan subnets: typically a `/24` per subnet (256 IPs, 251 usable)
3. Plan at least **one public subnet and one private subnet per Availability Zone** for high availability
4. For three AZs: 6 subnets minimum

**Draw this on paper first.**

### Step 2: Create the VPC and Subnets

1. In VPC console → Create VPC → Enter your CIDR block → Name it clearly (e.g., `prod-vpc`)
2. Create subnets:
   - Specify which AZ each belongs to
   - Specify CIDR range for each (e.g., `10.0.1.0/24` for Public-AZ1, `10.0.2.0/24` for Private-AZ1)
3. **Critical:** Subnets are private by default. Naming a subnet "Public" does NOT make it public — routing does.

### Step 3: Attach an Internet Gateway and Configure Routing

1. Create an Internet Gateway → Attach it to your VPC
2. Create a custom route table for public subnets:
   - Add route: Destination `0.0.0.0/0`, Target: Internet Gateway
   - Associate this route table with your public subnets
3. Public subnets now have internet access ✅

### Step 4: Configure NAT for Private Subnets

1. Create a NAT Gateway in a **public subnet** (requires an Elastic IP)
2. Create a route table for private subnets:
   - Add route: Destination `0.0.0.0/0`, Target: NAT Gateway
   - Associate with private subnets
3. Private instances can now initiate **outbound** internet traffic (patches, API calls) but cannot be reached inbound from the internet

---

## 5. CIDR Math Deep Dive

### Anatomy of `10.0.0.0/16`
10.0.0.0 /16
│
└── 16 bits fixed (network portion)
Remaining 16 bits variable (host addresses)
→ 2^16 = 65,536 total IPs


### Anatomy of `10.0.1.0/24`
10.0.1.0 /24
│
└── 24 bits fixed
Remaining 8 bits variable
→ 2^8 = 256 IPs (251 usable after AWS reserves 5)


### Public vs Private — It's All About the Route Table

| Route Table Type | Routes | What it enables |
|------------------|--------|-----------------|
| **Public Subnet Route Table** | `10.0.0.0/16 → local` (VPC internal) + `0.0.0.0/0 → IGW` | Two-way internet access |
| **Private Subnet Route Table** | `10.0.0.0/16 → local` + `0.0.0.0/0 → NAT Gateway` | Outbound internet only |
| **Isolated Subnet Route Table** | `10.0.0.0/16 → local` only | No internet at all |

---

## 6. Production 3-Tier VPC Architecture (Visual Mental Model)

This is the **standard architecture** for any production AWS workload: public subnets for internet-facing resources, private subnets for application tier and databases, across 2+ AZs for resilience.

VPC: 10.0.0.0/16 (eu-west-1)
│
├── Internet Gateway (IGW)
│
├── AZ-1a
│ ├── Public Subnet (10.0.1.0/24)
│ │ ├── Application Load Balancer
│ │ ├── Bastion Host (SSH entry point)
│ │ └── NAT Gateway
│ │
│ ├── Private Subnet - App Tier (10.0.2.0/24)
│ │ ├── EC2 App Servers
│ │ ├── ECS Tasks
│ │ └── Lambda ENIs
│ │
│ └── Private Subnet - Data Tier (10.0.3.0/24)
│ ├── RDS (Primary/Replica)
│ └── ElastiCache
│
└── AZ-1b (same structure, different IP ranges)
├── Public Subnet (10.0.4.0/24)
├── Private Subnet - App Tier (10.0.5.0/24)
└── Private Subnet - Data Tier (10.0.6.0/24)


**Security layers:**
- **Public subnets:** Security Group + NACL + IGW route
- **App subnets:** Security Group + NACL + NAT Gateway route (no inbound from internet)
- **Data subnets:** Security Group + NACL + **local route ONLY** (no internet of any kind)

---

## 7. Additional VPC Concepts

### VPC Peering

| What it is | A networking connection between two VPCs enabling private IP routing |
|------------|----------------------------------------------------------------------|
| Non-transitive rule | If VPC A peers with B, and B peers with C, **A cannot reach C through B** |
| To connect three VPCs | Need full-mesh peering (A-B, B-C, A-C) or use Transit Gateway |

### Elastic IP (EIP)

| What it is | A static, public IPv4 address you can attach to an EC2 instance or NAT Gateway |
|------------|-------------------------------------------------------------------------------|
| How it's different from auto-assigned public IP | Persists after stop/start (auto-assigned public IPs change) |
| Cost warning | You pay for **unattached** EIPs |

### VPC Endpoints

| What it does | Allows private communication between your VPC and AWS services without internet traffic |
|--------------|----------------------------------------------------------------------------------------|
| **Gateway endpoints** | S3 and DynamoDB only — **free** |
| **Interface endpoints** | Other AWS services — cost per hour |

---

## 8. Real-World Nigerian Scenario — Investment Platform

**Scenario:** A Nigerian retail investment platform (modelled on Cowrywise) manages mutual fund investments, fixed deposit products, and savings plans for 500,000 Nigerian retail investors. The SEC and CBN both require network isolation between customer-facing systems and financial processing backends.

### Architecture Decisions

**CIDR: `10.0.0.0/16`**
Choosing a `/16` gives 65,536 IPs — far more than needed today but leaves room for growth (loans, insurance, pensions). CIDR blocks cannot be easily changed after VPC creation.

**Two public subnets (one per AZ)**
Public subnets host only the Application Load Balancer and the Bastion Host. The ALB receives HTTPS traffic from retail investors. The Bastion Host is the **only** way engineers SSH into the environment. Security Group on the Bastion allows SSH only from the company's office static IP. This limits the public attack surface to two controlled entry points.

**Private application subnets**
The investment processing API runs on EC2 instances in private subnets. These instances have NO public IPs, no direct internet route. They receive traffic only from the ALB via the ALB's Security Group. Even if an attacker compromises the ALB, they cannot reach the API servers directly without also bypassing the SG rules.

**Private data subnets — completely isolated**
RDS (PostgreSQL) and ElastiCache live in private data subnets with route tables that have **NO `0.0.0.0/0` entry of any kind** — not even a NAT Gateway route. The only traffic these subnets handle is within the VPC. The Security Group on RDS allows connections only on port 5432 from the application servers' Security Group.

**Why no NAT Gateway route on data subnets?** It prevents any scenario where a compromised app server could use the database server as a jump box to reach the internet and exfiltrate data. No internet route = no exfiltration path.

---

## 9. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| **"Security Groups and NACLs work the same way"** | ❌ **Completely different.** SG = stateful (return traffic auto-allowed). NACL = stateless (need explicit outbound rules for ephemeral ports). |
| **"A subnet named 'Public' is automatically public"** | ❌ **No.** A subnet is public because its **route table** has a `0.0.0.0/0` route to an Internet Gateway. Naming means nothing. |
| **"NAT Gateway allows inbound internet to private subnets"** | ❌ **No.** NAT Gateway provides **outbound-only** internet for private subnets. Inbound requires public subnet + IGW + Load Balancer. |
| **"VPC peering is transitive"** | ❌ **No.** VPC peering is **non-transitive**. A→B and B→C does NOT mean A→C. Use Transit Gateway for transitive routing. |
| **"A subnet can span multiple Availability Zones"** | ❌ **Never.** Subnets are **zonal** — each subnet exists in exactly one AZ. VPCs span all AZs; subnets do not. |

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| What makes a subnet public? | Route table has `0.0.0.0/0 → IGW` |
| What makes a subnet private? | Route table has NO route to IGW |
| How do private instances reach the internet? | NAT Gateway in public subnet + route `0.0.0.0/0 → NAT` |
| Security Group — stateful or stateless? | **Stateful** (return traffic auto-allowed) |
| NACL — stateful or stateless? | **Stateless** (need explicit outbound rules) |
| Does a VPC span multiple AZs? | Yes — VPC is Regional |
| Does a subnet span multiple AZs? | No — subnet is zonal |
| VPC peering — transitive? | No — non-transitive |

---

## 11. One Sentence to Remember

> **A VPC is your isolated network spanning all AZs in a Region. Subnets are zonal — public if they route to an IGW, private if they don't. Security Groups are stateful (instance-level, allow only); NACLs are stateless (subnet-level, allow + deny). NAT Gateway gives private subnets outbound internet only.**

