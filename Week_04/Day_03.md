# ☁️ Week 4 · Day 3: EC2 Pricing Models

**Module:** Compute  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Here's a fact that surprises most people starting out in cloud: the most expensive thing about EC2 is almost never the wrong instance type. **It's the wrong pricing model.**

A `t3.medium` running On-Demand 24/7 costs about $30/month. The same instance on a 3-year Reserved Instance costs about $9/month. Same server. Same performance. **70% cheaper** — just by committing to a usage term.

Multiply that across dozens of servers = difference between a $5,000 AWS bill and a $15,000 bill. Same workload. Different choices.

**Why this matters in Nigeria:** Cloud cost overruns are the most common reason early-stage startups reduce AWS usage. Fintechs like Paystack, Flutterwave, and Kuda all use blended pricing strategies — Reserved for baseline, On-Demand or Spot for peaks.

---

## 2. The Core Principle

Match the pricing model to the workload's **predictability** and **flexibility**:

| Workload type | Model |
|---------------|-------|
| Predictable & always-on | Reserved or Savings Plans |
| Unpredictable spikes | On-Demand |
| Interruptible batch | Spot |
| Compliance / licensing | Dedicated |

**Mixing models is not cheating — it's professional cost engineering.**

---

## 3. The Nigerian Transport Analogy

| Transport | AWS Pricing Model |
|-----------|-------------------|
| Normal ride for a quick trip | On-Demand |
| Negotiated long-term arrangement | Reserved Instances |
| Cheapest option, may be interrupted | Spot Instances |
| Flexible bundle covering multiple transport types | Savings Plans |

---

## 4. On-Demand Instances

| Attribute | Detail |
|-----------|--------|
| **What it is** | Pay for compute by the second or hour. No commitment. |
| **Best for** | Dev/test, unpredictable traffic, new workloads, short-term experiments |
| **Avoid for** | Steady 24/7 workloads (you pay full price every hour) |
| **Billing** | Linux: per second (60s minimum). Windows: per hour. |
| **Capacity** | No capacity reservation by default |

---

## 5. Reserved Instances (RI)

A billing discount applied to On-Demand instances that match your RI's attributes. **NOT a physical server** — it's a commitment to pay in exchange for a discount.

| Term | Discount vs On-Demand |
|------|----------------------|
| 1-year | Up to 40% |
| 3-year | Up to 72% |

**Payment options:**
- All Upfront → most discount
- Partial Upfront
- No Upfront → least discount

**Use when:** Proven steady-state workloads running 24/7 (web servers, APIs, databases)

**Avoid when:** Workloads that might change size or be terminated — you pay whether you use it or not

### The Four Reserved Instance Types (Know All for Exam)

| Type | Discount | Flexibility | Capacity Reservation |
|------|----------|-------------|---------------------|
| **Standard RI** | Up to 72% | Cannot change family, OS, tenancy. Can change size within family. | No |
| **Convertible RI** | Up to 54% | CAN change family, OS, tenancy, size. Exchange for equal/greater value. | No |
| **Regional RI** | Full discount | Applies to any AZ in region automatically. Covers all sizes within family. | No |
| **Zonal RI** | Full discount | Tied to specific AZ. | **Yes** — guarantees capacity in that AZ |

**Exam critical:** Only Zonal RIs guarantee capacity. Standard and Regional RIs are billing discounts only — they don't reserve server space for you.

---

## 6. Spot Instances

AWS uses spare EC2 capacity that would otherwise go unused. You bid for that capacity at up to 90% off On-Demand price.

| Attribute | Detail |
|-----------|--------|
| **Max discount** | Up to 90% off On-Demand — cheapest EC2 option |
| **Interruption notice** | 2 minutes before AWS takes capacity back |
| **Use for** | Batch jobs, ML training, video rendering, data processing — fault-tolerant, interruptible workloads |
| **NEVER for** | Web servers, databases, anything that cannot tolerate sudden termination |
| **Spot Fleet** | Collection of Spot + On-Demand instances. Set target capacity and price ceiling. |

**Interruption handling:**
Poll `http://169.254.169.254/latest/meta-data/spot/termination-time` every 5 seconds. When notice appears, gracefully drain connections and checkpoint state.

---

## 7. Savings Plans

AWS's modern, more flexible replacement for Reserved Instances. Commit to a consistent amount of compute spend per hour — AWS automatically applies the maximum discount.

### Compute Savings Plans (Most Flexible)

| Attribute | Detail |
|-----------|--------|
| **Applies to** | EC2 + Fargate + Lambda |
| **Flexibility** | Any instance family, size, region, OS, or tenancy |
| **Discount** | Up to 66% |

### EC2 Instance Savings Plans

| Attribute | Detail |
|-----------|--------|
| **Locked to** | Specific instance family and region |
| **Flexible on** | Size, OS, tenancy within that family |
| **Discount** | Up to 72% |

### Savings Plans vs Reserved Instances

| Factor | Savings Plans | Reserved Instances |
|--------|---------------|-------------------|
| **Flexibility** | ✅ Wins — works across families, services, regions | Locked to specific attributes |
| **Max discount** | Up to 72% | Up to 72% (tie) |
| **Capacity reservation** | ❌ Cannot reserve capacity | ✅ Zonal RI can |
| **Simplicity** | ✅ Wins — no manual attribute matching | Requires matching instance types |

---

## 8. Dedicated Host vs Dedicated Instance

| | Dedicated Host | Dedicated Instance |
|--|----------------|-------------------|
| **What it is** | You rent an ENTIRE physical server | Instances run on hardware dedicated to your account |
| **Physical host control** | ✅ You control placement | ❌ You don't control which physical host |
| **License control** | ✅ BYOL for per-socket/core licenses (Oracle, SQL Server) | ❌ Cannot manage per-socket licensing |
| **Use for** | Software license terms require physical machine dedication | Compliance requires hardware isolation only |
| **Cost** | Charged per HOST per hour — most expensive | Charged per INSTANCE + small fee |

**Exam key:** Dedicated Instance = hardware isolation only. Dedicated Host = hardware isolation + license control.

---

## 9. Decision Framework — Match Workload to Model

| Workload Type | Best Model | Why |
|---------------|------------|-----|
| Dev/test, new, short-term | On-Demand | No commitment risk. Full flexibility. |
| Always-on production (known type) | Standard RI (3yr) | Max discount 72%. Worth it for stable workloads. |
| Always-on (type may change) | Convertible RI / Savings Plans | Flexibility to change as needs evolve. |
| Mix of EC2 + Fargate + Lambda | Compute Savings Plans | One commitment covers all compute services. |
| Batch jobs, ML, rendering | Spot Instances | Up to 90% off. Must tolerate interruption. |
| BYOL licensing (Oracle, SQL Server) | Dedicated Host | Physical server required for per-socket licensing. |
| Compliance isolation only | Dedicated Instance | Hardware isolation without license management. |

---

## 10. Real Cost Numbers (m6i.xlarge in eu-west-1)

| Pricing Model | Monthly Cost | Annual Cost | vs On-Demand |
|---------------|--------------|-------------|---------------|
| On-Demand | $140.16 | $1,682 | Baseline |
| 1yr RI — No Upfront | $91.98 | $1,104 | −35% |
| 1yr RI — All Upfront | $84.00 | $1,008 | −40% |
| 3yr RI — All Upfront | $48.00 | $576 | **−66%** |
| Spot Instance (avg) | ~$14.00 | ~$168 | **−90%** |
| Compute Savings Plan | ~$48–56 | ~$576–672 | −60 to −66% |

**Annual savings per m6i.xlarge:** $1,106 by switching from On-Demand to 3yr RI All Upfront — that's $92/month per server.

---

## 11. Real-World Scenario (TechStartNG)

**Challenge:** Digital lending platform paying $8,500/month in EC2 costs. CFO mandates 60% cost reduction without reducing performance.

### Audit Finding

4 `m6i.xlarge` running 24/7 at full On-Demand rates. 0% Reserved Instance coverage. $3,200/month on compute alone.

### Blended Pricing Architecture

| Tier | Setup | Annual Cost |
|------|-------|-------------|
| **App Tier (baseline)** | 2 `c6i.xlarge` → 3yr RI All Upfront | $1,152 |
| **App Tier (burst)** | 4 instances → On-Demand, scaled by ASG (~8hrs/day) | ~$500 |
| **Batch Tier** | Overnight processing (6hrs/night) → Spot Fleet | ~$66 |
| **Data Tier** | RDS Multi-AZ + Dedicated Host for Oracle BYOL | Separate |

### Results

| | Annual Cost |
|--|-------------|
| Before (all On-Demand) | $10,090 |
| After (blended strategy) | $2,339 |
| **Annual Savings** | **$7,751 (77% reduction)** |

---

## 12. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "RIs guarantee capacity" | ❌ RIs are **billing discounts** — NOT capacity reservations. Only Zonal RIs guarantee capacity. |
| "Excess instances get RI discount" | ❌ If you buy 30 RIs but launch 31 On-Demand — the 31st pays **full On-Demand price**. |
| "Spot is always wrong for production" | ❌ Spot IS appropriate for **stateless, fault-tolerant, interruptible** workloads in ASGs. Exam says "fault-tolerant" → consider Spot. |
| "Convertible RIs have higher discount than Standard" | ❌ Standard RI: up to 72%. Convertible RI: up to 54% (less discount for flexibility). |
| "RIs cover Fargate and Lambda" | ❌ RIs only cover EC2. **Savings Plans** cover EC2 + Fargate + Lambda. |
| "Dedicated Instance and Dedicated Host are the same" | ❌ Dedicated Instance = isolation only. Dedicated Host = isolation + physical host control + BYOL licensing. |

---

## 13. Practice Question (SAA-C03 Style)

**Scenario:** A Lagos fintech runs:
1. 4 application servers 24/7 for 3 years
2. Nightly 4-hour batch job (interruption-tolerant)
3. ML training twice per week for 8 hours

**Which TWO pricing models minimize cost?**

| Option | Answer |
|--------|--------|
| On-Demand for all three | ❌ Wrong — 24/7 servers over 3 years pay full price for 26,280 hours |
| **3-year Reserved Instances (Standard, All Upfront) for the 4 always-on servers** | ✅ Correct — 24/7 workload for 3 years = ideal Standard RI use case |
| Spot for the 4 application servers | ❌ Wrong — production servers cannot tolerate 2-minute interruptions |
| **Spot for BOTH nightly batch AND ML training** | ✅ Correct — both are interruptible, fault-tolerant, time-flexible |

---

## 14. Quick Reference Card

| Question | Answer |
|----------|--------|
| On-Demand discount | 0% (baseline) |
| 1-year RI discount | Up to 40% |
| 3-year RI discount | Up to 72% |
| Spot discount | Up to 90% |
| Compute Savings Plans applies to | EC2 + Fargate + Lambda |
| Which RI guarantees capacity? | Zonal RI only |
| Cheapest EC2 option | Spot (up to 90% off) |
| Best for BYOL (Oracle, SQL Server) | Dedicated Host |
| Best for mixed EC2 + Fargate | Compute Savings Plans |
| RIs are billing discounts or capacity? | Billing discounts (except Zonal RI) |

---

## 15. One Sentence to Remember

> **On-Demand for flexibility, Reserved (1yr/3yr up to 72%) for steady 24/7 workloads, Spot (up to 90%) for fault-tolerant batch jobs, Savings Plans for EC2+Fargate+Lambda mix, Dedicated Host for BYOL licensing. Mix models — that's professional cost engineering.**
