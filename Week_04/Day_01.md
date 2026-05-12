# ☁️ Week 4 · Day 1: EC2 Introduction & Launch

**Module:** Compute  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Amazon EC2 (Elastic Compute Cloud) is a virtual server in the AWS cloud. Instead of buying physical servers and waiting weeks, EC2 lets you provision a server in **under 90 seconds**, choose exact specs, and pay only for what you use.

**Why it matters:** Web servers, APIs, batch jobs, ML training — all run on EC2. RDS, ECS, and Lambda are built on top of EC2. It appears in Domains 1 (Resilient), 2 (Secure), and 4 (Cost-Optimized) on the exam.

**Nigerian context:** Paystack, Flutterwave, PiggyVest, Kuda all use EC2. It's the #1 topic in Lagos cloud interviews.

---

## 2. Core Concepts

### EC2 Instance

A virtual machine running on AWS hardware. You get a slice of CPU, RAM, and network. Runs inside a subnet in your VPC (Week 3 applies).

### AMI (Amazon Machine Image)

A template containing OS + pre-installed software.

| Source | What it is |
|--------|-----------|
| **AWS-provided** | Free official images (Amazon Linux, Ubuntu, Windows) |
| **AWS Marketplace** | Paid, pre-configured stacks |
| **Custom AMI** | Your own image from a configured instance |

**Why custom AMIs matter:** Configure once, create AMI, launch 100 identical copies in seconds. AMIs are region-specific — copy to another region before using there.

### Instance Type Naming

Pattern: `t3.micro` = family(t) + generation(3) + size(micro)

| Part | Meaning | Example |
|------|---------|---------|
| Family | Workload purpose | t, c, r, i, p |
| Generation | Hardware version | 3, 4, 5, 6 |
| Size | vCPU/RAM amount | nano, micro, small, large, xlarge |

### The Five Instance Families

| Family | Code | Best for |
|--------|------|----------|
| General Purpose | t, m | Web servers, dev (balanced CPU/RAM) |
| Compute Optimized | c | Batch processing, gaming, ML inference (high CPU) |
| Memory Optimized | r, x | Databases, caching (high RAM) |
| Storage Optimized | i, d | NoSQL, data warehousing (fast local disk) |
| Accelerated Computing | p, g | ML training, video rendering (GPU) |

**t3.micro is free tier eligible** — 2 vCPU, 1GB RAM.

---

## 3. Key Pair (SSH Access)

| What it is | Public-private key pair for authentication |
|------------|---------------------------------------------|
| **Critical rule** | Download `.pem` file **once** — AWS never shows it again |
| **Permission** | `chmod 400 my-key.pem` or SSH rejects it |
| **Lost key?** | **Cannot recover.** Must create new key pair and replace on instance. |

---

## 4. Storage Options

| Type | Persistence | Speed | Best for |
|------|-------------|-------|----------|
| **EBS gp3** | Survives stop/start | Good | Most workloads (default) |
| **EBS io2** | Survives stop/start | Very high (provisioned IOPS) | Databases |
| **EBS st1** | Survives stop/start | Good for sequential | Log processing |
| **EBS sc1** | Survives stop/start | Low | Infrequent access (cheapest) |
| **Instance Store** | **LOST** on stop/terminate | Extremely fast | Temporary data (caches, buffers) |

**Exam critical:** EBS persists. Instance Store is lost on stop or terminate. Never put permanent data on Instance Store.

---

## 5. Elastic IP (Static Public IP)

| What it is | Static public IPv4 address you allocate to your account |
|------------|--------------------------------------------------------|
| **Default behavior** | Public IP changes every stop/restart |
| **Elastic IP behavior** | Stays the same through stop/start cycles |
| **Cost** | Free while attached to running instance; charged ~$0.005/hr when unattached |

**Exam rule:** Production instances with DNS or IP allowlists → use Elastic IPs. Release unneeded EIPs or you pay for nothing.

---

## 6. User Data (Bootstrap Script)

A script that runs **once** when the instance first launches.

```bash
#!/bin/bash
apt-get update -y
apt-get install nginx -y
systemctl start nginx
```

## 7. EC2 Pricing Models (Heavily Tested)

| Model | Discount | Commitment | Best for | Never for |
|-------|----------|------------|----------|-----------|
| On-Demand | 0% | None | Dev/test, short spikes, first deployment | Steady 24/7 workloads |
| Reserved (1yr) | Up to 40% | 1 year | Steady-state production | Workloads that might change |
| Reserved (3yr) | Up to 72% | 3 years | Long-term steady workloads | Same as above |
| Spot | Up to 90% | None (interruptible) | Batch jobs, ML training, fault-tolerant work | Web servers, databases, anything stateful |
| Dedicated Host | Highest | N/A | Compliance, socket/core-based licenses (Oracle, SQL Server) | General workloads |

### Reserved Instance Types

| Type | Flexibility | Discount |
|------|-------------|----------|
| Standard RI | Locks instance type, size, AZ | Highest (up to 72%) |
| Convertible RI | Can change instance type, family, OS | Slightly less |
| Zonal RI | Specific AZ, **reserves capacity** | Same as standard |
| Regional RI | Any AZ in region, discount applies automatically | Same as standard |

**Key insight:** Standard RI = highest discount but locked in. Convertible RI = less discount but flexibility to change. Zonal RI = guarantees capacity in that AZ.

---

## 8. Stop vs Terminate vs Reboot (Critical Table)

| Action | CPU/RAM | Public IP | EBS Root | Billing |
|--------|---------|-----------|----------|---------|
| STOP | Released | **LOST** | Kept | Storage only (EBS continues) |
| START | Reallocated | **NEW IP** | Restored | Full billing resumes |
| TERMINATE | Destroyed | Lost | **DELETED** (by default) | All billing stops |
| REBOOT | Keeps same | Keeps same | Keeps same | Full billing continues |

**Critical facts:**

- Stop instance = stop paying for compute, but EBS storage keeps billing
- Change "Delete on Termination" to **NO** if you want EBS to persist after termination
- Public IP changes on every stop/start → use Elastic IP for static address
- Windows instances bill **per hour**; Linux bills **per second** (60s minimum)

---

## 9. Instance Metadata Service (IMDS)

Every EC2 instance can access its own metadata at `http://169.254.169.254/latest/meta-data/`

**What you can get:** instance ID, public IP, private IP, IAM role name, region, AMI ID — no credentials required.

```bash
curl http://169.254.169.254/latest/meta-data/instance-id
```
IMDSv2 is the newer, more secure version requiring a session token. The exam may ask about the security difference.

## 10. Real-World Scenario (Paystack-style Payment Platform)

**Context:** Payment API processing 4M requests/day. Peaks during market hours (9am–5pm) and salary days (25th–28th of month).

### Architecture

| Tier | Instance Type | Pricing | Why |
|------|--------------|---------|-----|
| Bastion Host | t3.micro | On-Demand | Used rarely. SSH from office IP only. |
| App servers (baseline) | c6i.xlarge | 1-year Reserved | 24/7 steady-state. Compute-heavy — crypto signing, fraud scoring, JSON processing. |
| App servers (burst) | c6i.xlarge | On-Demand | Auto Scaling adds during salary peaks, terminates after. |
| Batch jobs | c6i.xlarge | Spot | Overnight reconciliation. Can tolerate interruption. Saves up to 90%. |

### Optimizations

- Custom AMI with Node.js pre-baked → new instances ready in 45 seconds (vs 2-3 minutes installing via User Data)
- User Data only pulls latest config from S3
- Spread Placement Group across AZs → different hardware racks → one failure doesn't take both
- ALB in public subnets; app instances in private subnets (no public IPs)

---

## 11. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Public IP stays the same after stop/start" | ❌ Changes every stop/restart. Use Elastic IP for static address. |
| "Stopping an instance stops all billing" | ❌ EBS storage billing continues. CPU/RAM stops, but disks still bill. |
| "Linux and Windows both bill per second" | ❌ Linux = per second (60s min). Windows = per hour. |
| "Instance Store data survives stop/start" | ❌ Lost on stop or terminate. Use EBS for persistence. |
| "Reserved Instances reserve capacity" | ❌ RI is a billing discount. It doesn't guarantee capacity (except Zonal RI). |
| "Spot is fine for production web servers" | ❌ 2-minute termination notice. Never for stateful or real-time workloads. |
| "Lost key pair can be recovered from AWS" | ❌ Cannot recover. Must create new key pair. |
| "User Data runs on every start" | ❌ Runs once at first launch only. |

---

## 12. Quick Reference Card

| Question | Answer |
|----------|--------|
| Launch time | Under 90 seconds |
| Free tier eligible | t3.micro (2 vCPU, 1GB RAM) |
| Default storage | EBS gp3 |
| Survives stop/start? | EBS = Yes, Instance Store = No |
| Static public IP? | Elastic IP (free when attached) |
| Runs once at launch? | User Data (as root) |
| Cheapest for interruptible work? | Spot (up to 90% off) |
| Best for 24/7 production? | Reserved (up to 72% off) |
| Lost key pair? | Cannot recover — create new one |
| AMI region scope? | Region-specific — copy to use elsewhere |
| Linux billing | Per second (60s minimum) |
| Windows billing | Per hour |

---

## 13. One Sentence to Remember

> **EC2 = virtual servers in 90 seconds. AMI = OS template. Instance type = CPU/RAM. Key pair = SSH access (lost = gone forever). EBS persists; Instance Store doesn't. Public IP changes on stop; Elastic IP doesn't. User Data runs once at launch. On-Demand for short jobs, Reserved for 24/7, Spot for fault-tolerant batch, Dedicated Host for compliance.**

