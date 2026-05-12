
# ☁️ Week 4 · Day 2: EC2 Instance Types & AMIs

**Module:** Compute  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Amazon EC2 is AWS's core compute service — rent virtual servers on demand, pay per second (Linux) or per hour (Windows). Today covers two exam-critical concepts:

- **Instance Types** — the hardware profile (CPU, RAM, storage, network)
- **AMIs** — the OS + software snapshot used to launch configured servers

---

## 2. The Five EC2 Instance Families

AWS organizes instance types into five families. The instance name encodes everything.

| Family | Code | Optimized for | Example | Best for |
|--------|------|---------------|---------|----------|
| General Purpose | T / M | Balanced CPU/RAM | t3.micro, m6i.large | Web servers, dev/test, small databases |
| Compute Optimized | C | High CPU | c6i.xlarge, c5.2xlarge | Batch processing, video encoding, ML inference |
| Memory Optimized | R / X | Large RAM | r6i.large, x2idn.32xlarge | Redis, SAP HANA, big data analytics |
| Storage Optimized | I / D | High IOPS/throughput | i3.large, d3.8xlarge | NoSQL, data warehouses, distributed file systems |
| Accelerated Computing | P / G | GPU hardware | p3.2xlarge, g4dn.xlarge | ML training, graphics rendering, video transcoding |

---

## 3. Decoding an Instance Name

Every instance name follows a pattern: **Family · Generation · Processor · Size**

**Example: `m6i.2xlarge`**

| Part | Value | Meaning |
|------|-------|---------|
| Family | m | General Purpose |
| Generation | 6 | 6th generation (newer = better price/performance) |
| Processor | i | Intel Xeon |
| Size | 2xlarge | 8 vCPUs, 32 GiB RAM |

**Processor suffixes:**

| Suffix | Meaning |
|--------|---------|
| i | Intel Xeon |
| a | AMD |
| g | AWS Graviton (ARM) |
| n | AMD with NVMe |

---

## 4. Workload → Instance Selection Guide

Match the bottleneck to the family:

| Workload | Bottleneck | Family | Example |
|----------|-----------|--------|---------|
| Company website / API | Balanced | General (T/M) | t3.medium |
| Video transcoding | CPU | Compute (C) | c6i.2xlarge |
| Redis / MySQL analytics | RAM | Memory (R/X) | r6i.xlarge |
| PostgreSQL high IOPS | Disk I/O | Storage (I/D) | i3.large |
| ML model training | GPU | Accelerated (P/G) | p3.2xlarge |

**Exam tip:** T-series instances use CPU credits. They earn credits at a baseline rate and burst above it. For **sustained** high CPU, T-types will throttle — choose C-series instead.

---

## 5. What Is an AMI (Amazon Machine Image)?

An AMI is a complete template for launching a configured EC2 instance. It captures the OS, installed applications, configuration settings, and data — everything needed to reproduce a server exactly.

### What an AMI Contains

| Component | What it is |
|-----------|-----------|
| Root Volume Snapshot | Snapshot of the root EBS volume (OS + installed software) |
| Block Device Mapping | Defines which EBS volumes attach at launch |
| Launch Permissions | Controls which AWS accounts can use the AMI |
| Architecture | x86_64 (Intel/AMD) or arm64 (AWS Graviton) |

### Where AMIs Come From

| Source | What it is | Production Use? |
|--------|-----------|-----------------|
| **AWS Managed** | Official images (Amazon Linux 2, Ubuntu, Windows Server) | ✅ Yes — maintained and patched by AWS |
| **AWS Marketplace** | Pre-licensed third-party software (e.g., Windows + SQL Server) | ✅ Yes — billed through AWS |
| **Community AMIs** | Public, unverified images from other users | ❌ No — may contain malware |
| **Custom AMIs** | Built from your own configured instances | ✅ Yes — best practice for production |

### AMI Lifecycle

Launch Instance → Configure & Install Apps → Create AMI → Share/Copy to Regions → Deregister (stops new launches)

**Critical rule:** Deregistering an AMI prevents new launches but does **NOT** delete the underlying EBS snapshots. Delete snapshots separately to stop storage charges.

---

## 6. AMI vs EBS Snapshot (Exam Critical)

The exam frequently swaps these in answer choices. Know the difference cold.

| Attribute | AMI | EBS Snapshot |
|-----------|-----|---------------|
| **What it is** | Full server template | Point-in-time backup of a single volume |
| **Scope** | Entire EC2 instance (one or more volumes) | One EBS volume only |
| **Used to** | Launch new EC2 instances | Restore or clone a single volume |
| **Contains** | Snapshots + block device map + permissions + architecture | Raw block data only |
| **Region portability** | Copy AMI to new region, then launch | Copy snapshot independently |
| **Cost** | No charge for AMI registration | Charged per GB-month |
| **Deregister effect** | Does NOT delete underlying snapshots | Deleting breaks any AMI depending on it |

### When to Use an AMI

Launch multiple identical servers — Auto Scaling Groups, disaster recovery, replicating across accounts/regions.

### When to Use a Snapshot

Back up or clone a single volume — before risky changes, migrating a database, creating point-in-time restore points.

---

## 7. Real-World Nigerian Scenario (Konga-style)

**Problem:** E-commerce platform runs Laravel app on single `t3.medium`. Black Friday traffic spikes 20× — pages timeout, carts abandoned.

### Three-Step Solution

**Step 1: Right-Size the Instance**
Profile the app: CPU-bound (image processing, search) or memory-bound (product catalogs)?
- CPU-bound → `c6i.xlarge` (Compute Optimized)
- Memory-bound → `r6i.large` (Memory Optimized)

**Step 2: Create a Custom AMI**
Configure one instance perfectly: install Laravel, Nginx, PHP-FPM, set environment variables, apply security hardening. Create an AMI. Now launch 10 identical servers in 90 seconds — zero manual setup.

**Step 3: Auto Scaling + AMI**
Attach custom AMI to Auto Scaling Group. AWS launches new instances when CPU > 70%. After Black Friday, scales back down. Pay for peak demand only — not idle capacity year-round.

**Result:** Handles 20× traffic with zero downtime. Custom AMI + Auto Scaling is the standard AWS pattern for variable demand.

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "t3.micro is Free Tier — always choose it" | ❌ T-series throttles under sustained load. Choose C-series for sustained CPU workloads. Free Tier ≠ production suitability. |
| "You can change instance type while running" | ❌ Must **stop** EC2 instance before changing type. Stop → change type → start. |
| "Deregistering an AMI deletes its snapshots" | ❌ Deregistering does **NOT** delete snapshots. Snapshots continue accruing storage charges until manually deleted. |
| "Community AMIs are safe for production" | ❌ Not verified by AWS. May contain malware or misconfigurations. Use AWS Managed, Marketplace, or custom AMIs. |

---

## 9. Practice Question (SAA-C03 Style)

**Scenario:** A financial services company runs Redis (in-memory caching) and real-time fraud detection ML on the same EC2 instance. Fraud detection is frequently killed by OS due to memory pressure. They want separate, appropriately-sized instances.

**Which combination should be recommended?**

| Option | Answer |
|--------|--------|
| Two r6i.2xlarge instances | ❌ Redis is memory-bound, but ML inference benefits from GPU. Not optimal. |
| **One r6i.xlarge + One p3.2xlarge** | ✅ **CORRECT.** Redis → r6i (Memory). ML inference → p3 (GPU accelerated). |
| One m6i.large + One c6i.xlarge | ❌ m6i.large under-provisions Redis (needs large RAM). c6i.xlarge lacks GPU. |
| One t3.2xlarge for both workloads | ❌ T-series throttle under sustained load. Recreates original problem. |

**Key principle:** Match each workload to its bottleneck. Redis = memory → R-family. ML inference = GPU → P-family.

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| General Purpose family codes | T / M |
| Compute Optimized code | C |
| Memory Optimized codes | R / X |
| Storage Optimized codes | I / D |
| Accelerated Computing codes | P / G |
| What does "m6i.2xlarge" tell you? | Family(m) + Gen(6) + Processor(i=Intel) + Size(2xlarge) |
| T-series sustained load issue | Throttles — use C-series |
| What's in an AMI? | Snapshots + block mapping + permissions + architecture |
| AMI safest for production? | AWS Managed, Marketplace, or Custom (not Community) |
| Deregister AMI = delete snapshots? | ❌ No — delete snapshots manually |
| Change instance type requirement? | Stop instance first |

---

## 11. One Sentence to Remember

> **Match instance family to workload bottleneck: T/M for balanced, C for CPU, R/X for RAM, I/D for storage, P/G for GPU. Read instance names: family + generation + processor + size. AMI = full server template with OS + software + permissions. Deregistering AMI does NOT delete snapshots.**
