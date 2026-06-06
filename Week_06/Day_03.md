# ☁️ Week 6 · Day 3: Amazon Aurora Deep Dive

**Module:** Databases  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — What Makes Aurora Different

Amazon Aurora is AWS's cloud-native relational database, built from scratch to fix the architectural limits of standard MySQL/PostgreSQL on RDS. It is fully MySQL and PostgreSQL compatible — your existing applications connect without code changes. The difference is entirely under the hood: a completely redesigned storage layer that eliminates the EBS bottleneck.

### What Standard RDS Cannot Do

| Limitation | Standard RDS | Aurora |
|------------|--------------|--------|
| **Storage** | EBS-backed, provisioned upfront, grows manually | Auto-scales 10GB → 128TB in 10GB increments |
| **Read Replicas** | Up to 5, async lag | **Up to 15**, near-zero lag (under 10ms) |
| **Failover** | 1–2 minutes (standby warm-up) | **Under 30 seconds** (storage already shared) |
| **Storage replication** | Synchronous to standby only (Multi-AZ) | **6 copies across 3 AZs** always |
| **Throughput** | Standard engine performance | **5× MySQL, 3× PostgreSQL** |

**⚠️ Aurora supports MySQL and PostgreSQL only.** If your workload requires Oracle, SQL Server, or MariaDB → standard RDS.

---

## 2. Aurora Storage Architecture — The 6-Copy Revolution

Aurora's most significant innovation is the complete decoupling of compute and storage. Multiple DB instances — one writer and up to 15 readers — all connect to the same distributed storage volume.

### 6 Copies Across 3 AZs

| Metric | Value |
|--------|-------|
| Total copies | 6 (2 copies in each of 3 AZs) |
| Write quorum | 4 of 6 copies must confirm |
| Read quorum | 3 of 6 consistent copies |
| Resilience | Can lose 1 entire AZ and still write; lose 2 AZs and still read |

### Self-Healing Storage

- Continuously monitors all 6 storage segments
- Automatically repairs damaged segments using data from other 5 copies
- Runs entirely in background — database never pauses for repairs

### Shared Storage Advantage

| Benefit | Why |
|---------|-----|
| Near-zero replica lag | Readers receive changes via storage metadata, not log shipping (under 10ms) |
| Fast failover | Promote replica to writer with no data copy |
| Fast provisioning | New readers ready in seconds |
| Auto-scaling storage | 10GB → 128TB with no manual resize |

---

## 3. Aurora Cluster — Writer, Readers & Endpoints

An Aurora cluster is a set of DB instances sharing one distributed storage volume.

### Cluster Anatomy

| Component | Details |
|-----------|---------|
| **Writer Instance** | Handles ALL INSERT, UPDATE, DELETE. Exactly one writer per cluster. |
| **Reader Instances** | Up to 15, handle SELECT only. Can be in different AZs. |
| **Shared Storage** | 6 copies, 3 AZs, auto-scales 10GB→128TB, self-healing |

### Endpoints (Exam Critical)

| Endpoint | Purpose | Exam Clue |
|----------|---------|-----------|
| **Writer Endpoint** | Points to current writer. Auto-updates after failover. Enables zero-code failover. | Application writes → use Writer Endpoint |
| **Reader Endpoint** | Load-balances across all readers. Auto-refreshes when readers added/removed. | Application reads → use Reader Endpoint |
| **Custom Endpoint** | Points to specific subset of instances. Gives traffic shaping control. | "Route analytics to larger instances" or "isolate heavy queries" → **Custom Endpoint** |

---

## 4. Aurora vs RDS — Decision Framework

| Feature | Aurora | Standard RDS | Exam Rule |
|---------|--------|--------------|-----------|
| Engine compatibility | MySQL & PostgreSQL only | All 6 engines | Need Oracle/MSSQL → RDS |
| Storage model | Shared volume, auto 10GB→128TB | EBS volume, provisioned | Auto-scaling storage → Aurora |
| Read replicas | Up to 15, near-zero lag | Up to 5, async lag | >5 readers or low lag → Aurora |
| Failover speed | ~30 seconds | 60–120 seconds | Fast recovery SLA → Aurora |
| Storage replication | 6 copies across 3 AZs | Sync to standby only (Multi-AZ) | Aurora always has HA storage |
| Throughput | 5× MySQL, 3× PG | Standard | High throughput → Aurora |
| Serverless | Aurora Serverless v2 | Not available | Variable/unpredictable load → Aurora Serverless |
| Global database | <1 sec cross-region | Cross-region read replicas only | Global low-latency reads → Aurora Global |
| Cost | ~20% more than RDS | Lower base cost | Budget constrained, standard load → RDS |
| Backtrack | Rewind in place (no backup) | PITR only | "Undo changes without restore" → Aurora Backtrack |

---

## 5. Aurora Serverless v2 — Auto-Scaling Compute

Aurora Serverless v2 automatically adjusts compute capacity in real time based on actual workload demand.

### Key Features

| Feature | Detail |
|---------|--------|
| **ACU (Aurora Capacity Unit)** | 1 ACU ≈ 2GB RAM + proportional CPU/networking |
| **Scale increments** | As small as 0.5 ACU (very fine-grained) |
| **Scale-up speed** | Milliseconds — no pause in availability |
| **Scale down** | To configured minimum ACU (as low as 0.5 ACU) |
| **Pause?** | ❌ No — v2 does NOT pause (v1 did). Scales to minimum but remains available. |
| **Billing** | Per ACU-hour of actual consumption |
| **Mixed cluster** | Provisioned + serverless instances in same cluster (writer provisioned, readers serverless) |
| **Full feature support** | Works with Global Database, read replicas, Multi-AZ, Performance Insights |

### Ideal Use Cases

- Dev/test environments
- New applications with unknown load
- Salary-day style load spikes
- Multi-tenant SaaS with variable tenant activity

**Storage costs are separate and continue regardless of compute scaling.**

---

## 6. Aurora Global Database — Cross-Region in Under 1 Second

Replicates an entire Aurora cluster to secondary AWS Regions at the storage layer — achieving **under 1 second lag globally**.

### Architecture

| Component | Details |
|-----------|---------|
| **Primary Region** | Full cluster (writer + up to 15 readers). All writes happen here. |
| **Secondary Regions** | Up to 5 read-only Aurora clusters. Each has its own reader instances. |
| **Replication** | Storage-layer, under 1 second lag |
| **DR promotion** | Any secondary region can be promoted to primary in under 1 minute |

### Benefits

| Benefit | Why |
|---------|-----|
| Local low-latency reads | Users query local secondary cluster |
| Independent read capacity | Does not compete with primary region |
| Managed DR | No manual data copy for failover |

**Exam clue:** "Serve users in multiple continents with low latency" or "cross-region DR with under 1 minute RTO" → Aurora Global Database.

---

## 7. Real-World Scenarios

### QuickPay NG, Lagos — Payment Platform

**Challenge:** 3M transactions/day. Traffic 20× higher on salary day. Read traffic 15× write traffic.

**Solution:**

| Component | Configuration |
|-----------|--------------|
| Writer Instance | All payment writes via Writer Endpoint |
| 10 Reader Instances | Balance checks, transaction history via Reader Endpoint |
| 2 Analytics Readers | Custom Endpoint routes heavy report queries to dedicated large instances |
| Serverless v2 | Salary day overflow — auto-scales to absorb 20× spike |
| Backtrack | Rewind in-place if engineer runs UPDATE without WHERE |

### AgroLend, Abuja — Agricultural Finance

**Challenge:** 800,000 farmers. Traffic near-zero in planting season (June-Aug), 50× higher during harvest loan assessment (Oct-Nov). Need <50ms latency in Lagos, Accra, Nairobi.

**Solution:**

| Component | Configuration |
|-----------|--------------|
| Serverless v2 | Scales 0.5→64 ACU — near-zero cost off-season, full capacity at harvest |
| Global Database | Primary in eu-west-1 (London) |
| Secondary clusters | af-south-1 (Cape Town serving Ghana/Lagos), me-south-1 (Bahrain serving Nairobi) |
| DR failover | Under 1 minute if primary region fails |

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Aurora architecture is same as standard RDS" | ❌ Aurora uses **distributed shared storage (6 copies, 3 AZs)**. RDS uses EBS. |
| "Aurora Read Replicas have same lag as RDS" | ❌ Aurora replica lag is **under 10ms** (storage metadata, not log shipping). RDS lag can be seconds to minutes. |
| "Backtrack restores from S3 backup" | ❌ **Backtrack rewinds live DB in place** — no backup, no restore time. Only Aurora has this. |
| "Aurora Serverless v2 pauses at zero traffic" | ❌ v2 **scales to minimum ACU (0.5) but does NOT pause**. Storage costs continue. v1 had pause; v2 removed it. |
| "Aurora supports Oracle and SQL Server" | ❌ **MySQL and PostgreSQL only**. For Oracle/SQL Server → standard RDS. |

---

## 9. Knowledge Check — Practice Questions

### Q1 — Analytics Isolation

**Scenario:** Aurora cluster has 1 writer + 12 readers. Analytics team runs monthly reports (4 hours) that slow main application.

**Correct answer:** **Create a Custom Endpoint pointing to 2 dedicated large reader instances for analytics only**

| Option | Why wrong/right |
|--------|-----------------|
| Increase writer instance size | ❌ Doesn't help reads |
| **Custom Endpoint for analytics** | ✅ Routes analytics traffic to dedicated readers — app unaffected |
| Migrate analytics to separate RDS | ❌ Adds cost and complexity |
| Enable Multi-Master | ❌ For writes, not analytics isolation |

### Q2 — Fastest Recovery

**Scenario:** Engineer runs `UPDATE transactions SET amount = 0 WHERE 1=1` — zeroes 2M records. Error discovered 20 minutes later.

**Correct answer:** **Use Aurora Backtrack to rewind the database to 19 minutes ago in place**

| Option | Why wrong/right |
|--------|-----------------|
| Restore from S3 backup | ❌ Takes 3–4 hours |
| Point-in-Time Recovery | ❌ Works but requires new cluster + switchover — much slower |
| **Backtrack** | ✅ Rewinds live DB in minutes — no restore, no downtime |
| Promote pre-error Read Replica | ❌ Would need replica that predates error (unlikely) |

### Q3 — Cross-Region Low Latency

**Scenario:** Need <50ms latency for users in Lagos, Accra, Nairobi. Write traffic only from Lagos.

**Correct answer:** **Aurora Global Database with primary in eu-west-1 and secondary clusters in af-south-1 and me-south-1**

| Option | Why wrong/right |
|--------|-----------------|
| Multi-AZ with cross-AZ replicas | ❌ Doesn't span regions |
| Serverless v2 in all three regions | ❌ No cross-region replication |
| **Aurora Global Database** | ✅ <1 sec storage-layer replication, local reads in each region |
| Cross-region RDS Read Replicas | ❌ Higher lag, fewer features |

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| Aurora storage model | 6 copies across 3 AZs, self-healing |
| Write quorum | 4 of 6 copies |
| Read quorum | 3 of 6 copies |
| Max Aurora read replicas | 15 (vs RDS: 5) |
| Aurora replica lag | <10ms (storage metadata, not log shipping) |
| Aurora failover time | ~30 seconds (vs RDS: 1–2 min) |
| Aurora engines | MySQL & PostgreSQL only |
| Aurora Backtrack | Rewinds live DB in place — no restore |
| Serverless v2 scaling | Milliseconds, no pause, min 0.5 ACU |
| Global Database lag | <1 second across regions |
| Global Database DR promotion | <1 minute |
| Custom Endpoint use | Route analytics to isolated readers |

---

## 11. One Sentence to Remember

> **Aurora = shared 6-copy storage across 3 AZs (5× MySQL, 3× PostgreSQL, 15 readers, <30 sec failover). Serverless v2 scales compute in milliseconds (no pause, min 0.5 ACU). Global Database = <1 sec cross-region replication, <1 min DR promotion. Backtrack rewinds in place. Custom Endpoints isolate traffic (analytics). Aurora = MySQL/PG only — for Oracle/SQL Server use RDS.**
