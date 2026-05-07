# ☁️ Week 3 · Day 2: Subnets — Public vs Private

**Module:** Networking & VPC  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

A subnet is where your actual resources live — EC2, RDS, Lambda, ECS. The subnet you choose determines: internet access, security controls, which AZ the resource sits in, and whether it's reachable from outside your VPC.

**The public/private subnet decision is the most fundamental architectural choice in AWS networking.**

---

## 2. Core Concepts

### Public Subnet

| What makes it public | Route table has `0.0.0.0/0 → Internet Gateway (IGW)` |
|---------------------|-----------------------------------------------------|
| What can live here | Load balancers, bastion hosts, NAT gateways |
| Internet access | Two-way (inbound + outbound) |

### Private Subnet

| What makes it private | Route table has NO direct route to an IGW |
|----------------------|-------------------------------------------|
| What can live here | Application servers, databases, caches |
| Internet access | Outbound only (via NAT Gateway) or none at all |

### The 5 Reserved IPs (AWS reserves these in EVERY subnet)

| IP | Purpose |
|----|---------|
| .0 | Network address |
| .1 | VPC router |
| .2 | DNS server |
| .3 | Future use (AWS reserved) |
| .255 | Broadcast |

**A `/24` subnet (256 total) has 251 usable IPs.** Always subtract 5.

### Subnet AZ Binding

| Rule | Subnet = exactly ONE AZ. Cannot span AZs. |
|------|-------------------------------------------|
| For high availability | Create separate subnets in each AZ, deploy resources across them |

### Auto-Assign Public IP

| What it is | Subnet-level setting that automatically gives EC2 instances a public IP |
|------------|-----------------------------------------------------------------------|
| Private subnets | MUST be DISABLED |
| Public subnets | Enable only for instances that need direct internet access |

---

## 3. Default VPC vs Custom VPC (Critical for Exam)

| | Default VPC | Custom VPC |
|--|-------------|------------|
| **Subnet type** | ALL subnets are **public** | Subnets are **private by default** |
| **Auto-assign IP** | Enabled | Disabled by default |
| **CIDR** | Fixed (172.31.0.0/16) | You choose |
| **Use case** | Learning only | **All production workloads** |

**Never use Default VPC for production.** Every subnet in Default VPC has an IGW route + auto-assign IP = every EC2 gets a public IP unless you block it with a Security Group.

---

## 4. The 3-Tier Subnet Pattern (Industry Standard)

| Tier | Subnet type | Route table | What lives here |
|------|-------------|-------------|-----------------|
| **Web/Public** | Public | `0.0.0.0/0 → IGW` | Load Balancer, Bastion Host, NAT Gateway |
| **App/Private** | Private | `0.0.0.0/0 → NAT Gateway` | EC2 app servers, ECS tasks, Lambda |
| **Data/Isolated** | Private | `LOCAL ONLY` (no internet route) | RDS, ElastiCache |

**Multi-AZ example (6 subnets):**
VPC: 10.0.0.0/16

AZ-1a AZ-1b
├── Public: 10.0.1.0/24 ├── Public: 10.0.2.0/24
├── Private-App: 10.0.11.0/24 ├── Private-App: 10.0.12.0/24
└── Private-Data: 10.0.21.0/24 └── Private-Data: 10.0.22.0/24


---

## 5. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "A subnet named 'Public' is public" | ❌ **Routing determines publicness, not naming.** No IGW route = private. |
| "A subnet can span multiple AZs" | ❌ **Never.** Subnets are zonal. VPCs span AZs; subnets don't. |
| "/24 subnet has 256 usable IPs" | ❌ **251 usable.** AWS reserves 5 IPs per subnet. |
| "Default VPC is secure by default" | ❌ **Default subnets are ALL public.** Never use for production. |
| "A Security Group can make a public subnet private" | ❌ **No.** SG blocks traffic but doesn't remove the IGW route. Compliance requires the route table to have no internet entry. |

---

## 6. Real-World Nigerian Scenario — Payment Gateway (Flutterwave-style)

**Requirement:** PCI-DSS + CBN compliance — cardholder data systems must have NO internet path.

| Tier | Subnet type | Route table | Why |
|------|-------------|-------------|-----|
| Public | Public | `0.0.0.0/0 → IGW` | ALB + WAF only. No data here. |
| Private-App | Private | `0.0.0.0/0 → NAT` | Payment processing microservices. Outbound only. |
| Private-Data | **Isolated** | `LOCAL ONLY` (no NAT route) | RDS + ElastiCache. **Zero internet path** — provable for auditors. |

**Key insight:** The route table itself is the audit evidence. A Security Group blocking traffic is NOT enough — the route table must have no `0.0.0.0/0` entry.

---

## 7. Quick Reference

| Question | Answer |
|----------|--------|
| What makes a subnet public? | Route table has `0.0.0.0/0 → IGW` |
| What makes a subnet private? | No IGW route (may have NAT route for outbound) |
| What makes a subnet isolated? | Local route ONLY — no internet route at all |
| How many usable IPs in a /24? | 251 (256 total - 5 reserved) |
| Can a subnet span multiple AZs? | No |
| Should you use Default VPC for production? | Never |
| What proves compliance to auditors? | The route table (not Security Group rules) |

---

## 8. One Sentence to Remember

> **A subnet is public only if its route table has an Internet Gateway route. Default VPC subnets are all public — never use for production. AWS reserves 5 IPs per subnet. Subnets are zonal; VPCs are Regional.**
