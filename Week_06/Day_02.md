# ☁️ Week 6 · Day 2: Amazon RDS Deep Dive

**Module:** Databases  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — What Is Amazon RDS?

Amazon RDS (Relational Database Service) is AWS's fully managed relational database service, supporting **six database engines**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora.

**"Managed" means AWS handles:**
- Hardware provisioning
- OS installation and patching
- Engine patching
- Automated backups
- Hardware monitoring
- Automatic host replacement

**You focus on:** schema design, data quality, query optimization, and application logic.

---

## 2. Key Concepts — Definitions to Memorize

| Term | Definition |
|------|------------|
| **RDS** | Fully managed relational DB. AWS handles hardware, OS patching, backups. You manage schema, data, users, queries. |
| **Multi-AZ** | Synchronous standby replica in different AZ. Automatic DNS failover if primary fails (~1-2 min). Standby is **NOT readable**. Zero data loss (synchronous replication). |
| **Read Replica** | Asynchronous, read-only copy of primary DB. Offloads read traffic. Can be promoted to standalone DB. Up to 5 replicas; cross-region supported. |
| **PITR (Point-In-Time Recovery)** | Restore DB to any second within retention window (0-35 days). Powered by daily snapshots + transaction logs every 5 minutes. Creates **new** DB instance. |
| **Storage Types** | gp3 SSD (default) for most workloads. io1/io2 for high-throughput, low-latency. Storage Auto Scaling expands automatically when free space < 10%. |

---

## 3. Multi-AZ vs Read Replicas — The Most-Tested Distinction

This appears on nearly every SAA-C03 exam. Know this table cold.

| Feature | Multi-AZ | Read Replica |
|---------|----------|--------------|
| **Purpose** | HIGH AVAILABILITY & FAILOVER | READ SCALING & PERFORMANCE |
| **Replication** | Synchronous (zero data loss) | Asynchronous (slight lag possible) |
| **Standby Readable?** | ❌ NO — failover only | ✅ YES — application reads from it |
| **Auto-Failover?** | ✅ YES — DNS flips in ~1-2 min | ❌ NO — manual promotion only |
| **Cross-Region?** | No — same region, different AZ | Yes — up to 5 replicas any region |
| **Write Performance** | Unchanged (writes to both) | Unchanged (writes to primary only) |
| **Exam Trigger Words** | "automatic failover", "high availability", "no data loss", "DR within region" | "improve read performance", "offload reporting", "scale read traffic", "global users" |

**⚠️ Critical Rule:** If the exam asks about improving read performance, **Multi-AZ is always wrong**. Use Read Replicas instead. The Multi-AZ standby is NEVER readable.

---

## 4. Amazon Aurora — Cloud-Native Relational Database

Aurora is AWS's flagship cloud-native relational database — compatible with MySQL and PostgreSQL, but with a completely redesigned distributed storage engine.

### Performance

| Metric | Value |
|--------|-------|
| MySQL throughput | **5×** standard MySQL |
| PostgreSQL throughput | **3×** standard PostgreSQL |
| Max INSERT/s | 600,000 |
| Max UPDATE/s | 200,000 |

### Self-Healing Distributed Storage

| Feature | Detail |
|---------|--------|
| **Replication** | 6 copies across 3 AZs (2 copies per AZ) |
| **Auto-healing** | Damaged storage segments repaired automatically |
| **Storage growth** | 10 GB → 128 TB automatically (10 GB increments) |
| **Manual provisioning** | Not needed |

### Aurora Serverless v2

| Feature | Detail |
|---------|--------|
| **Compute scaling** | 0.5–128 Aurora Capacity Units, zero downtime |
| **Billing** | Per second of capacity used |
| **Best for** | Variable/unpredictable workloads, dev/test, seasonal spikes |

### Aurora Global Database

| Feature | Detail |
|---------|--------|
| **Structure** | 1 primary region + up to 5 read-only secondary regions |
| **Replication lag** | < 1 second |
| **RPO / RTO** | < 1 second / < 1 minute |
| **Best for** | Global applications, cross-region DR |

### Aurora vs Standard RDS — Quick Comparison

| Feature | Standard RDS | Aurora |
|---------|--------------|--------|
| Read Replicas | Up to 5 | Up to 15 |
| Failover Time | ~1–2 minutes | ~30 seconds |
| Storage | Fixed, provisioned | Auto to 128 TB |
| Database Engines | All 6 engines | MySQL & PG only |
| Storage Copies | Standard replication | 6 copies across 3 AZs |
| Serverless Option | No | Serverless v2 |

**📌 Exam Keys:**
- Aurora = **MySQL & PostgreSQL only**. If question mentions Oracle, SQL Server, or MariaDB → standard RDS.
- "Unpredictable workloads" → Aurora Serverless v2.
- "Cross-region DR, RTO < 1 minute" → Aurora Global Database.

---

## 5. Security, Encryption & Backups

### Security Controls

| Control | Detail |
|---------|--------|
| **VPC Isolation** | Runs inside your VPC. DB subnet group must span ≥2 AZs. Never expose DB endpoint to internet. |
| **Security Groups** | Control inbound access to DB port (5432/3306). Never `0.0.0.0/0` in production. |
| **IAM DB Authentication** | Short-lived IAM tokens instead of passwords (MySQL & PostgreSQL only). No DB credentials in code. |
| **RDS Proxy** | Connection pool between Lambda/serverless apps and RDS. Reduces connections by up to 99%. Exam trigger: "Lambda connecting to RDS" or "too many DB connections." |

### Backups & Recovery

| Backup Type | Retention | Deleted with DB? |
|-------------|-----------|------------------|
| **Automated Backups** | 0–35 days (daily snapshots + transaction logs every 5 min) | ✅ YES — tied to instance lifecycle |
| **Manual Snapshots** | Until you delete them | ❌ NO — persist after DB deletion |

### Point-In-Time Recovery (PITR)

- Restore to any second within retention window
- Creates **new** DB instance (does not overwrite original)
- Recovery time: typically 5–30 minutes

### ⚠️ Encryption Trap (Top Exam Question)

You **cannot** enable encryption on an existing unencrypted DB.

**Required 3-step process:**
1. Take a snapshot of the unencrypted DB
2. Copy the snapshot with encryption enabled
3. Restore a new encrypted DB instance from the encrypted copy

---

## 6. Real-World Nigerian Scenario — PayFast Fintech, Lagos

### The Challenge

| Problem | Impact |
|---------|--------|
| 500,000 active customers on aging on-premises Oracle DB | Performance bottlenecks |
| Two 30-minute outages last quarter | Lost revenue, customer complaints |
| Heavy reporting queries slow live payment processing | Transaction delays |
| 3 DBAs spend 80% time on maintenance | Product development suffers |

### The Solution

| Step | Action | Benefit |
|------|--------|---------|
| 1 | **RDS PostgreSQL with Multi-AZ** | Automatic failover in ~1 minute (vs 30+ minutes) |
| 2 | **3 DBAs → DBA burden drops from 80% to ~15%** | AWS handles patching, backups, monitoring |
| 3 | **Add 2 Read Replicas** | Offload reporting queries — read capacity improves 3× |
| 4 | **14-day automated backups** | CBN 7-year compliance (PITR to any second) |
| 5 | **Storage Auto Scaling** | Database grows 200 GB → 2 TB automatically |

### Results

| Metric | Before | After |
|--------|--------|-------|
| DBA maintenance | 80% | ~15% |
| Failover time | 30+ minutes | ~1 minute |
| Read capacity | Baseline | 3× improvement |
| Compliance | Manual | Automated PITR |

---

## 7. Exam Traps — Memorize These

| Trap | Truth |
|------|-------|
| **"Multi-AZ standby is readable"** | ❌ Standby exists only for automatic failover — never serves read traffic. For read performance → Read Replicas. **Most-tested trap.** |
| **"Automated backups persist after DB deletion"** | ❌ Automated backups are **deleted with the DB**. Take a manual snapshot before deleting. |
| **"You can encrypt an existing unencrypted RDS instance"** | ❌ No toggle. Required: snapshot → copy encrypted → restore new instance. |
| **"Aurora supports all 6 database engines"** | ❌ Aurora = **MySQL & PostgreSQL only**. Oracle/SQL Server/MariaDB → standard RDS. |
| **"Read Replicas provide automatic failover"** | ❌ Promotion is **manual**. Multi-AZ = automatic failover. |

**💡 Bonus Rule:** Multi-AZ = availability. Read Replicas = read scaling. They solve different problems. Never mix them up.

---

## 8. Practice Question (SAA-C03 Style)

**Scenario:** A Nigerian fintech runs their payment platform on Amazon RDS MySQL in a single-AZ setup. They had two 30-minute outages last quarter and report that read-heavy reporting queries are slowing down live transaction processing.

**Which combination of changes should a Solutions Architect recommend?**

| Option | Answer |
|--------|--------|
| **Enable Multi-AZ + add a Read Replica** | ✅ **Correct** — Multi-AZ solves outages (auto-failover). Read Replica solves read performance (offload reporting). |
| Enable Multi-AZ — standby will handle both failover and reporting reads | ❌ Multi-AZ standby is **NOT readable**. Most common trap. |
| Add two Read Replicas — one will automatically become primary if current fails | ❌ Read Replicas do **NOT** provide auto-failover. Promotion is manual. |
| Migrate to Aurora — automatically resolves both HA and read performance | ❌ Aurora doesn't auto-configure Multi-AZ or Read Replicas. Migration is more complex than enabling existing RDS features. |

**Answer:** B — Multi-AZ solves HA; Read Replica solves read scaling.

---

## 9. Quick Reference Card

| Question | Answer |
|----------|--------|
| Multi-AZ purpose | High Availability (automatic failover) |
| Read Replica purpose | Read scaling (offload read traffic) |
| Multi-AZ standby readable? | ❌ No |
| Read Replica auto-failover? | ❌ No (manual promotion) |
| Max RDS Read Replicas | 5 |
| Max Aurora Read Replicas | 15 |
| Aurora engines | MySQL & PostgreSQL only |
| Aurora vs RDS storage | Aurora auto to 128TB |
| Aurora Serverless v2 best for | Unpredictable/variable workloads |
| Aurora Global Database RTO | < 1 minute |
| Encrypt existing unencrypted RDS? | Snapshot → copy encrypted → restore |
| RDS Proxy best for | Lambda + RDS (connection pooling) |

---

## 10. One Sentence to Remember

> **RDS = managed relational DB (6 engines). Multi-AZ = High Availability (synchronous, automatic failover, standby NOT readable). Read Replicas = read scaling (asynchronous, up to 5, manual promotion). Aurora = cloud-native (MySQL/PG only, 5× performance, 15 replicas, 128TB auto-storage, Serverless v2, Global Database). Cannot encrypt existing unencrypted DB — snapshot → copy encrypted → restore.**
