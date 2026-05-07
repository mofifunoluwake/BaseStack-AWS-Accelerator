# ☁️ Week 3 · Day 3: Route Tables, Internet Gateways & NAT

**Module:** Networking & VPC  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Route tables are the traffic control system of your VPC. Every packet is evaluated against a route table, which answers: "where should this traffic go?" Without understanding route tables, you cannot debug connectivity problems or design secure network segmentation.

---

## 2. Core Concepts

### Route Table

| What it is | A set of rules (routes) that determine where network traffic from a subnet is directed |
|-------------|---------------------------------------------------------------------------------------|
| Binding | Each subnet associates with **exactly one** route table |
| Route parts | Destination CIDR (e.g., `0.0.0.0/0`) + Target (e.g., `igw-xxx`, `nat-xxx`, `local`) |

### Route Entry Anatomy

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | `local` | VPC-internal traffic stays inside |
| `0.0.0.0/0` | `igw-xxx` | Internet traffic goes to Internet Gateway |
| `0.0.0.0/0` | `nat-xxx` | Internet traffic goes to NAT Gateway (outbound only) |

### Most Specific Route Wins (Longest Prefix Match)

When a packet matches multiple routes, the **narrowest** (most specific) route wins.
Packet to 10.0.1.5:
├── 10.0.1.0/24 → local ← WINNER (/24 is most specific)
├── 10.0.0.0/16 → local ← loses (/16 is less specific)
└── 0.0.0.0/0 → igw-xxx ← loses (catch-all, least specific)


### The Local Route (Sacred & Permanent)

| What it is | `VPC CIDR → local` — automatically added to EVERY route table |
|------------|---------------------------------------------------------------|
| Can you delete it? | **No** — permanent and undeletable |
| What it means | VPC-internal traffic NEVER leaves the VPC (even if `0.0.0.0/0 → IGW` exists) |

### Internet Gateway (IGW)

| What it does | Enables **bidirectional** internet for public subnets |
|--------------|------------------------------------------------------|
| What makes a subnet public | Route table has `0.0.0.0/0 → igw-xxx` |
| Three requirements for internet | IGW attached to VPC + route table has IGW route + instance has public IP |

### NAT Gateway

| What it does | Enables **outbound-only** internet for private subnets |
|--------------|--------------------------------------------------------|
| Where it lives | In a **public subnet** (requires Elastic IP) |
| What it blocks | All unsolicited inbound traffic |
| Cost | $0.045/hour + $0.045/GB processed |
| HA requirement | One NAT Gateway **per AZ** (cross-AZ NAT = single point of failure) |

### Main Route Table

| What it is | Default route table created with every VPC |
|-------------|---------------------------------------------|
| What happens to subnets without explicit association | They use the **Main Route Table** |
| Best practice | Keep Main Route Table as **LOCAL ONLY** (no IGW/NAT routes) |

---

## 3. Route Table Rules (Exam Critical)

| Rule | Explanation |
|------|-------------|
| **One route table per subnet** | A subnet cannot have multiple route tables |
| **Many subnets per route table** | One route table can serve many subnets |
| **Most specific route wins** | Longest prefix match determines routing |
| **Local route is permanent** | Cannot be deleted or overridden |
| **Unassociated subnets = Main RT** | Dangerous default — be intentional |

---

## 4. Standard Route Table Setup

| Route Table | Routes | Associated subnets |
|-------------|--------|-------------------|
| **public-rt** | `10.0.0.0/16 → local` + `0.0.0.0/0 → IGW` | Public subnets (ALB, Bastion, NAT) |
| **private-rt** | `10.0.0.0/16 → local` + `0.0.0.0/0 → NAT` | Private-app subnets (EC2, ECS) |
| **isolated-rt** | `10.0.0.0/16 → local` ONLY | Private-data subnets (RDS, ElastiCache) |

---

## 5. Traffic Flow Examples

### User → ALB (Inbound Internet)
Internet → IGW → Public Subnet (ALB) → Security Group allows HTTPS

### ALB → EC2 App (Internal VPC)
ALB (10.0.1.x) → EC2 (10.0.11.x)
Uses local route (10.0.0.0/16 → local) — never leaves VPC

### EC2 App → Internet (Outbound via NAT)
EC2 (10.0.11.x) → NAT Gateway (in public subnet) → IGW → Internet
Private-rt has 0.0.0.0/0 → NAT Gateway

### EC2 App → RDS (Internal, Isolated)
EC2 (10.0.11.x) → RDS (10.0.21.x)
Uses local route. Isolated-rt has NO 0.0.0.0/0 entry.


---

## 6. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "NAT Gateway is free" | ❌ **$0.045/hour + $0.045/GB**. For HA: 2 NAT Gateways = double cost. |
| "IGW alone gives internet access" | ❌ Need **all three**: IGW attached + route table entry + public IP on instance. |
| "Subnet with no route table has no routing" | ❌ It uses the **Main Route Table** by default. |
| "Local route can be overridden" | ❌ **Permanent.** VPC-internal traffic always stays local. |
| "NAT Gateway works for all protocols" | ❌ **TCP, UDP, ICMP only** — no GRE, no IPsec. |
| "One NAT Gateway per VPC is HA" | ❌ NAT GW is **AZ-scoped**. If that AZ fails, NAT fails. Deploy one **per AZ**. |

---

## 7. Real-World Nigerian Scenario (OPay-style)

**Finding 1:** Main Route Table had `0.0.0.0/0 → IGW` — 4 subnets without explicit association were publicly routable.

**Fix:** Removed IGW route from Main Route Table. Created explicit route tables for all subnets.

**Finding 2:** Single NAT Gateway in one AZ serving both AZs — cross-AZ dependency created single point of failure.

**Fix:** Deployed second NAT Gateway in other AZ. Created per-AZ private route tables.

---

## 8. Quick Reference

| Question | Answer |
|----------|--------|
| What makes a subnet public? | Route table has `0.0.0.0/0 → IGW` |
| What makes a subnet private? | No IGW route (may have NAT route) |
| What makes a subnet isolated? | Local route ONLY — no `0.0.0.0/0` entry |
| Can you delete the local route? | No |
| What does unassociated subnet use? | Main Route Table |
| NAT Gateway cost? | $0.045/hour + $0.045/GB |
| NAT Gateway HA requirement? | One per AZ |

---

## 9. One Sentence to Remember

> **Route tables determine public vs private — most specific route wins, local route is permanent. NAT Gateway costs money, requires one per AZ for HA, and supports only TCP/UDP/ICMP. Subnets without explicit association use the Main Route Table (keep it local-only).**
