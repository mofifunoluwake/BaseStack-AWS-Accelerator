# ☁️ Day 4: Cloud Economics — How AWS Saves You Money

**Module:** Cloud Foundations  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

Traditional IT is financially painful. You buy servers for your *peak* demand — even if that peak lasts only 3 days a year. The other 362 days, that $50,000 server sits mostly idle. That's called **CapEx** (Capital Expenditure) — spending big money before you make any revenue.

Cloud flips the model to **OpEx** (Operational Expenditure) — you pay only for what you use, per second or per request. No idle hardware. Cash flow improves overnight.

**For a Lagos startup:** You can launch a production-ready application on AWS with ₦50,000 in credits. You scale your spend *as* your customers grow. Paystack, Flutterwave, and Piggyvest all built this way.

---

## 2. The 5 Pricing Models (Memorise These for the Exam)

### On-Demand (0% discount, but zero commitment)

| What it is | Best for | Exam warning |
|------------|----------|--------------|
| Pay per second/hour/request. No upfront cost. No termination fees. | Dev/test, unpredictable workloads, short-term experiments | This is your baseline pricing — everything else is compared to On-Demand |

**Analogy:** Paying for a hotel room by the night. Expensive per night, but you can leave anytime.

---

### Reserved Instances (up to 72% discount)

| What it is | Best for | Exam warning |
|------------|----------|--------------|
| Commit to 1 or 3 years of using a specific instance type in a specific Region | Steady-state, predictable workloads (e.g., a web server that runs 24/7) | You're locked to an instance family (e.g., m5.large) — less flexible than Savings Plans |

**Analogy:** Signing a 1-year apartment lease. Cheaper per month than a hotel, but you're locked in.

**Payment options for Reserved Instances:**
- **All Upfront** — biggest discount, pay once
- **Partial Upfront** — lower upfront, smaller discount
- **No Upfront** — pay monthly, smallest discount

---

### Savings Plans (up to 66% discount)

| What it is | Best for | Exam warning |
|------------|----------|--------------|
| Commit to a *dollar amount per hour* of compute spend, not a specific instance type | Flexible compute across EC2, Lambda, and Fargate | **Two types:** Compute Savings Plans (maximum flexibility) vs EC2 Instance Savings Plans (less flexible) |

**Analogy:** A prepaid card that works at any restaurant in a food court, not just one specific stall.

**Why Savings Plans are better than Reserved Instances for some workloads:**
- You can change instance families (t3 → m5 → c6g)
- You can change Regions
- It covers Lambda and Fargate too

---

### Spot Instances (up to 90% discount)

| What it is | Best for | Exam warning |
|------------|----------|--------------|
| Rent spare AWS capacity at huge discounts — but AWS can take it back with 2 minutes' notice | Fault-tolerant, interruptible workloads (batch jobs, data processing, containerised stateless apps) | **NEVER for:** Primary databases, stateful applications, anything that can't be interrupted |

**Analogy:** Standby airline tickets. Crazy cheap, but you might get bumped off the flight.

**Good Spot use cases:**
- Nightly log processing (if interrupted, just restart)
- CI/CD build runners
- Containerised stateless microservices

**Bad Spot use cases:**
- Production database
- User session state
- Real-time payment processing

---

### AWS Free Tier (100% discount — free)

| What it is | Best for | Exam warning |
|------------|----------|--------------|
| Three types of free usage limits | Learning, experimentation, small prototypes | **Memorize the three types** — this is an exam question |

**The Three Free Tier Types (KNOW THESE):**

| Type | Duration | Examples |
|------|----------|----------|
| **12-month free** | First 12 months after account creation | EC2 t2.micro (750 hrs/month), S3 (5GB), RDS (750 hrs) |
| **Always free** | No expiration — forever | Lambda (1M requests/month), CloudWatch (10 metrics), SNS (1M publishes) |
| **Short-term trials** | Usually 1–2 months | Redshift, QuickSight, SageMaker |

**Key insight for the exam:**  
- S3 free tier = 12 months (not always free)  
- Lambda free tier = always free (not limited to 12 months)  
- EC2 free tier = 12 months (not always free)

---

## 3. Data Transfer Costs (The Sneaky Exam Topic)

Many people assume "same Region = all free." That's wrong.

| Data transfer scenario | Is it free? |
|------------------------|-------------|
| Inbound data to AWS (uploading) ✅ | Always free |
| Between EC2 and S3 in same Region ✅ | Free |
| Between EC2 instances in **same AZ** ✅ | Free |
| Between EC2 instances in **different AZs** (same Region) ❌ | Small charge (~$0.01/GB) |
| Between different Regions ❌ | Larger charge |
| Outbound to public internet ❌ | Largest charge (~$0.09–$0.15/GB) |

**Exam trap:** When you see an architecture with EC2 instances across two AZs for high availability, remember — inter-AZ data transfer *will* appear on the bill.

---

## 4. Real-World Nigerian Scenario: Paystack (Before vs After)

### ❌ BEFORE: Physical Servers

| Cost item | Amount |
|-----------|--------|
| Server purchase (production-grade) | $50,000 |
| Data centre colocation fees (per year) | $15,000 |
| Dedicated sysadmin salary | $8,000 |
| **Total Year 1** | **$73,000+** |

Plus: Fixed capacity — servers idle 90% of the time. Black Friday? You already bought enough capacity. Quiet Tuesday afternoon? Still paying the same.

### ✅ AFTER: AWS

| Cost item | Amount |
|-----------|--------|
| Monthly AWS bill (starting) | ~$300/mo |
| **Total Year 1** | **~$3,600** |

Plus: Auto-scaling — pay per transaction. Minimal cost at 3am. Scales up for Black Friday automatically. AWS manages the infrastructure — team focuses on product.

**Savings:** ~$69,400 in Year 1 alone.

---

## 5. How to Never Get a Surprise Bill

| Tool | What it does | When to use |
|------|--------------|-------------|
| **AWS Pricing Calculator** | Estimate costs BEFORE you build | Before launching any new architecture |
| **AWS Budgets** | Set alerts for actual or forecasted spend | **Set a ₦0 budget alert on Day 1** — it will warn you if you're about to be charged |
| **AWS Cost Explorer** | Visualize historical spend, 13-month forecast | Monthly review of where your money went |

**Pro tip for the exam:** When a question asks about *preventing* surprise bills, the answer is **AWS Budgets** (alerts) or **AWS Cost Explorer** (analysis).

---

## 6. Exam Traps

| Wrong belief | Actual truth |
|--------------|---------------|
| "All data transfer within a Region is free" | ❌ Inter-AZ transfer has charges. |
| "Free Tier is only EC2" | ❌ Three types covering EC2, S3, Lambda, RDS, and more. |
| "Reserved Instances give the biggest discount" | ❌ Spot gives up to 90% (vs 72% for RIs). But Spot has interruption risk. |
| "Savings Plans and Reserved Instances are the same" | ❌ Savings Plans are more flexible (any instance family, any Region, covers Lambda/Fargate). |
| "Spot Instances are fine for my primary database" | ❌ NEVER. Two-minute termination notice = data loss risk. |

---

## 7. Which Pricing Model When? (Quick Decision Guide)

| What does the workload need? | Choose this |
|------------------------------|-------------|
| "Guaranteed availability, no interruptions, short-term" | On-Demand |
| "Runs 24/7 for years, predictable" | Reserved Instances or Savings Plans |
| "Fault-tolerant batch job, want cheapest possible" | Spot Instances |
| "Want flexibility to change instance families/Regions" | Compute Savings Plans |
| "Learning AWS, want to pay nothing" | AWS Free Tier |
| "Primary production database" | On-Demand or Reserved — NEVER Spot |

---

## 8. One Sentence to Remember

> **On-Demand for guaranteed availability, Spot for dirt-cheap batch jobs, Reserved/Savings Plans for steady 24/7 workloads, and never assume all same-Region traffic is free — inter-AZ transfer costs money.**
