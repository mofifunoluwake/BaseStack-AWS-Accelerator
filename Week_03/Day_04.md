# ☁️ Week 3 · Day 4: Security Groups & NACLs

**Module:** Networking & VPC  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

AWS gives you **two separate firewall layers** in your VPC:

| Layer | What it is | Level |
|-------|-----------|-------|
| **Security Group (SG)** | Bouncer at each apartment door | Instance-level (EC2, RDS, Lambda) |
| **Network ACL (NACL)** | Security desk at building entrance | Subnet-level (entire subnet) |

Traffic passes through **both** layers. Both must allow it.

---

## 2. Security Groups (Instance-Level Firewall)

| Feature | Behavior |
|---------|----------|
| **Stateful** | Return traffic is **automatic** — no outbound rule needed |
| **Rules** | Allow rules **only** (no explicit Deny) |
| **Evaluation** | ALL rules evaluated — most permissive wins |
| **Default inbound** | DENY all (nothing allowed until you add rules) |
| **Default outbound** | ALLOW all |

### SG Referencing (The Secure Database Pattern)

Instead of using an IP address as source, use another Security Group ID.
sg-app-servers (EC2) → allows port 5432 → sg-rds (RDS)


**Why this is better:** When EC2 instances scale out, new instances are in `sg-app-servers`, so they automatically get database access — no rule changes needed.

### Security Group Rule Example

| Type | Protocol | Port | Source | Action |
|------|----------|------|--------|--------|
| SSH | TCP | 22 | 10.0.100.0/28 | ALLOW (Bastion only) |
| HTTPS | TCP | 443 | 0.0.0.0/0 | ALLOW (ALB inbound) |
| PostgreSQL | TCP | 5432 | sg-app-servers | ALLOW (app servers only) |
| All | All | All | — | DENY (implicit) |

---

## 3. Network ACLs (Subnet-Level Firewall)

| Feature | Behavior |
|---------|----------|
| **Stateless** | Return traffic **NOT automatic** — need explicit outbound rules |
| **Rules** | Allow AND Deny rules |
| **Evaluation** | Number order — **first match wins** (lowest number first) |
| **Default NACL** | ALLOW ALL inbound + outbound |
| **Custom NACL** | DENY ALL inbound + outbound (only `*` DENY rule) |

### The Ephemeral Port Trap (Most Common NACL Mistake)

When a client connects to a server, the OS picks a random temporary port (1024–65535) for the response to come back on.

**Because NACLs are stateless:** You must explicitly allow outbound traffic on ports 1024–65535 for responses to leave your subnet.

**Security Groups don't have this problem** — they're stateful and handle return traffic automatically.

### NACL Rule Example (Numbered)

| # | Protocol | Port | Source/Dest | Action |
|---|----------|------|-------------|--------|
| 100 | TCP | 443 | 0.0.0.0/0 | ALLOW (inbound HTTPS) |
| 110 | TCP | 80 | 0.0.0.0/0 | ALLOW (inbound HTTP) |
| 120 | TCP | 1024-65535 | 0.0.0.0/0 | ALLOW (ephemeral responses OUT) |
| * | All | All | 0.0.0.0/0 | DENY (implicit catch-all) |

---

## 4. Traffic Flow Through Both Layers
Internet Traffic
↓
┌─────────────────────────────────────┐
│ LAYER 1: NACL (Subnet boundary) │
│ Stateless — checks EVERY packet │
│ First matching rule wins │
└─────────────────────────────────────┘
↓ (if allowed)
┌─────────────────────────────────────┐
│ LAYER 2: Security Group (Instance) │
│ Stateful — remembers connection │
│ All rules evaluated, most permissive│
└─────────────────────────────────────┘
↓ (if allowed)
INSTANCE
↓ (response)
┌─────────────────────────────────────┐
│ Security Group (auto-allowed) │
│ Stateful → no outbound rule needed │
└─────────────────────────────────────┘
↓
┌─────────────────────────────────────┐
│ NACL (must have outbound rule) │
│ Needs explicit ephemeral port allow│
└─────────────────────────────────────┘


---

## 5. Security Group vs NACL — Side by Side (Exam Critical)

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Level** | Instance (ENI) | Subnet |
| **State** | **STATEFUL** | **STATELESS** |
| **Rule types** | Allow only | Allow + Deny |
| **Evaluation** | All rules — most permissive | Number order — first match |
| **Default (new)** | Deny inbound, Allow outbound | Default = Allow all; Custom = Deny all |
| **Ephemeral ports** | Not needed | **REQUIRED** (1024–65535) |
| **Can source be SG?** | ✅ YES (SG referencing) | ❌ NO (CIDR blocks only) |
| **Best use** | Primary security layer | Secondary defence (block specific IPs) |

---

## 6. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Add a Deny rule to Security Group to block an IP" | ❌ **SG has no explicit Deny.** Use NACL Deny rule instead. |
| "NACL rules are evaluated all at once like SG" | ❌ **Number order** — lower number wins. A Deny at rule 90 beats an Allow at rule 100. |
| "Custom NACL works like default NACL" | ❌ **Custom = DENY ALL** by default. You must add ALL Allow rules manually. |
| "NACL handles return traffic automatically" | ❌ **Stateless** — you must allow ephemeral ports (1024–65535) outbound. |
| "If SG looks right, traffic should work" | ❌ **Check both SG AND NACL.** Either can block traffic. Use VPC Flow Logs to see which. |

---

## 7. Real-World Nigerian Scenario (Interswitch-style)

**Finding 1:** SSH (port 22) open to `0.0.0.0/0` on all EC2 instances

**Fix:** Remove `0.0.0.0/0`. Add rule: SSH from `sg-bastion` only. Engineers SSH to Bastion first, then jump to internal servers.

**Finding 2:** RDS allowed traffic from entire VPC CIDR (`10.0.0.0/16`)

**Fix:** Replace CIDR with SG reference: port 5432 from `sg-app-servers` only. Now only app servers can query the database.

**Finding 3:** Custom NACL on data subnet was silently dropping responses

**Root cause:** Custom NACL started with DENY ALL. Inbound allowed port 5432, but outbound didn't allow ephemeral ports (1024–65535) for responses.

**Fix:** Added outbound rule: TCP 1024–65535 to app subnet CIDR → ALLOW. Timeouts resolved immediately.

---

## 8. Quick Reference

| Question | Answer |
|----------|--------|
| Stateful or stateless? | SG = Stateful, NACL = Stateless |
| Need outbound rule for return traffic? | SG = No, NACL = Yes (ephemeral ports) |
| Can you Deny a specific IP? | SG = No, NACL = Yes |
| Rule evaluation order? | SG = All rules, NACL = Number order (first match) |
| Default inbound? | SG = Deny all, Default NACL = Allow all |
| Custom NACL default? | Deny all (only `*` rule) |
| Can source be another SG? | SG = Yes, NACL = No (CIDR only) |
| Ephemeral port range? | 1024–65535 (must be in NACL outbound) |

---

## 9. One Sentence to Remember

> **Security Groups are stateful (instance-level, allow only, return traffic auto-allowed). NACLs are stateless (subnet-level, allow+deny, first match wins, need explicit ephemeral port rules for responses). Both must allow traffic.**
