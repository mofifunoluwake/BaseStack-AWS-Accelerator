# ☁️ Week 6 · Day 1: Database Selection on AWS

**Module:** Databases  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why Database Selection Is Critical

Database selection is the most consequential architecture decision you make.

| Wrong choice | Right choice |
|--------------|--------------|
| Performance bottlenecks | Effortless scaling |
| Data integrity failures | Milliseconds queries |
| Costs 10× higher | Predictable bill |

**What AWS provides:** 15+ purpose-built database services, each optimized for a specific data model and access pattern. The SAA-C03 exam tests your ability to match a scenario to the right service.

| Stat | Value |
|------|-------|
| AWS DB services | 15+ |
| Cost difference (wrong vs right) | Up to 10× |
| Exam coverage (5 core services) | 95% of questions |

---

## 2. The 5 Core AWS Database Services

These five services cover 95% of SAA-C03 database questions.

| Service | Type | Best For | Key Differentiator | Exam Weight |
|---------|------|----------|-------------------|-------------|
| **RDS** | Relational (OLTP) | Complex queries, joins, ACID | 6 engine options | ⭐⭐⭐⭐ |
| **Aurora** | Relational (OLTP) | High-throughput, global apps | 5× MySQL speed, 128TB auto-scale | ⭐⭐⭐⭐⭐ |
| **DynamoDB** | NoSQL Key-Value | Millions ops/sec, flexible schema | Single-digit ms at any scale | ⭐⭐⭐⭐⭐ |
| **ElastiCache** | In-Memory Cache | Sub-ms reads, session data | NOT primary DB — cache only | ⭐⭐⭐⭐ |
| **Redshift** | Data Warehouse (OLAP) | Petabyte analytics, BI dashboards | Columnar storage, Spectrum | ⭐⭐⭐⭐ |

**Also on SAA-C03 (appear less frequently):**
- **DocumentDB** — MongoDB-compatible document DB
- **Neptune** — Graph DB for connected data (social networks, fraud detection)
- **Keyspaces** — Apache Cassandra-compatible

---

## 3. Relational Deep Dive — RDS & Aurora

Both services are ACID compliant and support the relational model with complex joins. The key question: do you need multiple engines (RDS) or AWS's proprietary performance engine (Aurora)?

### Amazon RDS — Managed Relational Database

| Feature | Detail |
|---------|--------|
| **Engines** | MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, Aurora |
| **Multi-AZ** | Synchronous standby in another AZ — automatic failover for HA. Standby cannot serve reads. |
| **Storage** | Auto-scales 10GB → 128TB across 3 AZs, 6 copies |
| **Read Replicas** | Async copies for read scaling (up to 5). For performance, NOT for HA. |
| **Backups** | Point-in-time recovery up to 35 days |
| **Use when** | Existing SQL apps, Oracle/SQL Server licensing, moderate load |

### Amazon Aurora — AWS-Optimized Relational Engine

| Feature | Detail |
|---------|--------|
| **Performance** | 5× MySQL / 3× PostgreSQL throughput |
| **Storage** | Auto-scales to 128TB across 3 AZs, 6 copies — self-healing |
| **Read Replicas** | **15** replicas with sub-10ms lag (vs 5 for standard RDS) |
| **Aurora Serverless** | Auto-pauses when idle — unpredictable workloads |
| **Aurora Global Database** | Cross-region replication with sub-1-second lag |
| **Use when** | High throughput, global apps, cost matters at scale, green-field projects |

**Exam Critical:** Multi-AZ = High Availability (automatic failover), NOT for read performance. For read scaling, use **Read Replicas**. This distinction appears on nearly every RDS/Aurora question.

---

## 4. NoSQL & Caching — DynamoDB + ElastiCache

### Amazon DynamoDB — NoSQL Key-Value + Document

| Feature | Detail |
|---------|--------|
| **Architecture** | Items stored by Partition Key (mandatory) + optional Sort Key. No table joins. Data denormalized by design. |
| **Performance** | Single-digit ms reads/writes at ANY scale — 1 req/sec to 10 million/sec |
| **Serverless** | No servers, no capacity planning. Pay per request or provisioned capacity. |
| **DAX** | DynamoDB Accelerator — in-memory cache for microsecond latency (read-heavy) |
| **Global Tables** | Multi-region, multi-master replication. Writes to any region, replicated globally in under 1 second. |
| **Use when** | Gaming leaderboards, product catalogs, session data, IoT, flexible schema |

### Amazon ElastiCache — In-Memory Cache (NOT a Primary Database)

ElastiCache sits in front of your primary database (RDS or DynamoDB) and caches frequently accessed data in RAM for sub-millisecond reads. **Must always be backed by a persistent database.**

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Persistence** | Optional | No — pure cache |
| **Pub/Sub messaging** | ✅ | ❌ |
| **Data structures** | Sorted sets, geospatial queries | Simple key-value |
| **High availability** | Multi-AZ with automatic failover | No failover |
| **Transactions** | ✅ | ❌ |
| **Multi-threaded** | No | Yes |
| **Best for** | Complex caching, session store, queues | Simple, high-throughput cache |

**⚠ Critical:** ElastiCache is a cache, NOT a primary database. Data can be evicted or lost. If a question says "needs to persist data permanently" → ElastiCache is wrong.

---

## 5. OLTP vs OLAP — The Critical Distinction

This is the **most tested concept** in the SAA-C03 database section. Every database question implicitly asks: is this transactional (OLTP) or analytical (OLAP)?

| | OLTP | OLAP |
|--|------|------|
| **What it handles** | Many small, fast transactions in real time | Complex queries across huge historical datasets |
| **Operations** | INSERT, UPDATE, DELETE, SELECT (individual rows) | GROUP BY, aggregations, joins across billions of rows |
| **Data shape** | Current/live data — row-oriented storage | Historical data — columnar storage |
| **AWS services** | RDS, Aurora, DynamoDB | **Redshift** |
| **Examples** | Payment processing, order placement, banking | Monthly sales reports, BI dashboards, trend analysis |

### Exam Signal Keywords

| 🔵 OLTP Signals | 🟣 OLAP Signals |
|----------------|-----------------|
| Payment / order / booking | Petabyte / data warehouse |
| INSERT / UPDATE millions/sec | Analytics / reporting / BI |
| Real-time transaction | Historical trends / dashboards |
| ACID compliance | Columnar storage |
| Complex joins | Aggregations / GROUP BY |

---

## 6. Real-World Nigerian Scenario — Konga-Style E-Commerce

A Nigerian e-commerce startup (think Konga or Jumia) has 4 distinct data workloads. This is **polyglot persistence** — multiple databases in one architecture.

| Workload | Service | Why |
|----------|---------|-----|
| **Order & Payment Processing** | **Aurora (PostgreSQL)** | ACID transactions non-negotiable for payments. 5× throughput over RDS at same cost. Multi-AZ for zero downtime during peak. |
| **Product Catalog & User Sessions** | **DynamoDB** | 60,000+ SKUs with different attributes — flexible schema. Handles flash sales (millions reads/sec) without pre-provisioning. |
| **Homepage / Cart (Hot Data Cache)** | **ElastiCache Redis** | Trending section and active carts load in <10ms. Caches DynamoDB reads. Protects primary DB during traffic spikes. |
| **BI & Sales Reports** | **Redshift** | Monthly sales reports, vendor analytics, demand forecasting across 2 years of history. Petabyte-scale OLAP queries that would cripple Aurora. |

**Discussion prompt:** Think of a Nigerian bank, hospital, or government agency. Which of their workloads maps to which AWS database service?

---

## 7. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| **"Multi-AZ improves read performance"** | ❌ Multi-AZ = **High Availability** (automatic failover). Standby cannot serve reads. For read performance → Read Replicas. |
| **"ElastiCache can be the primary database"** | ❌ ElastiCache is a **cache**. Data can be evicted or lost. Must always be backed by persistent DB (RDS/DynamoDB). |
| **"DynamoDB replaces RDS when joins are needed"** | ❌ DynamoDB does **not** support complex joins or cross-table relational integrity. Use RDS/Aurora for ACID + joins. |
| **"Redshift is for OLTP transactions"** | ❌ Redshift is **OLAP** columnar data warehouse. NOT for high-frequency small INSERTs/UPDATEs. Use Aurora/RDS for OLTP. |

---

## 8. Practice Question (SAA-C03 Style)

**Scenario:** A Nigerian fintech processes 40,000 payment transactions per second. Each transaction requires ACID compliance and cross-table joins between customer accounts and ledger records. Transaction volume expected to triple in 12 months.

**Which database BEST meets these requirements?**

| Option | Answer |
|--------|--------|
| DynamoDB with Global Tables and DAX | ❌ DynamoDB doesn't support complex joins or cross-table relational integrity |
| **Aurora (PostgreSQL) with Multi-AZ, Read Replicas, and Aurora Global Database** | ✅ **Correct** — ACID + joins = relational. Aurora Global for scale. 5× throughput handles 3× growth. |
| RDS MySQL with cross-region Read Replicas | ❌ Cross-region Read Replicas have significant lag, not designed for global ACID at this scale |
| Redshift with multi-node cluster | ❌ Redshift is OLAP — wrong for real-time transactional payment processing |

**Answer:** B — ACID + joins = relational. High throughput + global scale = Aurora over standard RDS.

---

## 9. Quick Reference Card

| Question | Answer |
|----------|--------|
| Relational with complex joins | RDS or Aurora |
| NoSQL, flexible schema, massive scale | DynamoDB |
| In-memory cache, sub-ms reads | ElastiCache (backed by persistent DB) |
| Petabyte analytics, BI dashboards | Redshift |
| Multi-AZ purpose | High availability (failover), not read performance |
| Read scaling for RDS | Read Replicas (async, up to 5) |
| Read scaling for Aurora | Read Replicas (async, up to 15, sub-10ms lag) |
| Aurora vs RDS advantage | 5× throughput, 15 replicas, global database |
| DynamoDB DAX purpose | In-memory cache for microsecond reads |
| ElastiCache Redis vs Memcached | Redis = persistence, Pub/Sub, HA; Memcached = simple, multi-threaded |

---

## 10. One Sentence to Remember

> **RDS/Aurora for relational (ACID + joins), DynamoDB for NoSQL scale (flexible schema, millions ops/sec), ElastiCache for cache (backed by persistent DB, never primary), Redshift for OLAP analytics (columnar, petabyte-scale). Multi-AZ = HA (not read performance). OLTP = transactions (RDS/Aurora/DynamoDB). OLAP = analytics (Redshift).**
