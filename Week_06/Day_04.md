# ☁️ Week 6 · Day 4: Amazon DynamoDB Deep Dive

**Module:** Databases  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why DynamoDB?

DynamoDB is the database that powered Amazon Prime Day — 300 million shopping cart operations per hour, single-digit millisecond latency, without changing a line of code.

### Where Relational Breaks Down

| Problem | Why |
|---------|-----|
| **Vertical scaling ceiling** | Relational databases scale up (add CPU/RAM) — one server has a physical limit |
| **Complex joins** | At Prime Day scale, joins become catastrophically slow |
| **Table locks** | A single query can lock entire tables, causing cascading failures |

**The hard truth:** No single server — regardless of cost — can handle hundreds of millions of operations per hour with sub-10ms latency.

### DynamoDB's Answer

| Feature | Benefit |
|---------|---------|
| **Horizontal scaling** | Automatically splits data across many servers (partitions). No physical ceiling. |
| **Serverless** | No servers to provision, no storage to configure, no engine to patch |
| **Guaranteed latency** | Single-digit milliseconds at any scale — 1 user or 100 million |
| **Automatic partitioning** | Splits and distributes data as table grows |
| **Proven at scale** | Prime Day, Airbnb, Lyft, Duolingo, Samsung, Snapchat |

| Stat | Value |
|------|-------|
| Prime Day ops/hour | 300M |
| Max throughput | 10M requests/second |
| Latency guarantee | Single-digit ms |

---

## 2. Table Anatomy — Items, Keys & Schema

### Core Vocabulary

| Term | Definition | SQL Equivalent |
|------|------------|----------------|
| **Table** | Collection of items | Table |
| **Item** | One record | Row |
| **Attribute** | A piece of data on an item | Column (but optional) |
| **Primary Key** | Uniquely identifies each item | Primary Key |

**Key difference:** Items in the same table do NOT need identical attributes. Schema flexibility is a feature, not a bug.

### Primary Key Types

| Type | Structure | Example |
|------|-----------|---------|
| **Simple Primary Key** | Partition Key only | `UserID` → lookup one user |
| **Composite Primary Key** | Partition Key + Sort Key | `CustomerID` (PK) + `OrderDate` (SK) |

**How it works:**
- **Partition Key** determines which physical partition stores the item (hash function)
- **Sort Key** (if present) allows sorting and range-querying items within the same partition

### Sample Users Table — No Enforced Schema

| UserID (PK) | Name | Email | City | Age |
|-------------|------|-------|------|-----|
| user_001 | Emeka Eze | emeka@gmail.com | Lagos | 28 |
| user_002 | Amaka Obi | amaka@yahoo.com | Abuja | — |
| user_003 | Tunde Bakare | tunde@outlook.com | Port Harcourt | 35 |

Notice `user_002` is missing the Age attribute — perfectly valid in DynamoDB.

---

## 3. Indexes — GSI & LSI (Query Flexibility)

DynamoDB can only query efficiently by Primary Key. To search by other fields, you need an Index.

### LSI — Local Secondary Index

| Attribute | Detail |
|-----------|--------|
| **Structure** | Same Partition Key, different Sort Key than main table |
| **Creation** | Must be created at table creation — cannot be added later |
| **Capacity** | Shares capacity with main table (no extra billing) |
| **Max count** | 5 LSIs per table |
| **Example** | Table: PK=CustomerID, SK=OrderDate. LSI: SK=TotalAmount → query "Emeka's orders sorted by amount" |

### GSI — Global Secondary Index

| Attribute | Detail |
|-----------|--------|
| **Structure** | Completely different Partition Key (and optional Sort Key) |
| **Creation** | Can be created anytime — even after table creation |
| **Capacity** | Has its own capacity (separate billing) |
| **Max count** | 20 GSIs per table |
| **Example** | Table: PK=CustomerID. GSI: PK=ProductID → query "all customers who bought ProductX" |

### When to Use Which

| Index | Use Case |
|-------|----------|
| **LSI** | Alternate sort order within the same partition (e.g., orders by amount instead of date, same customer) |
| **GSI** | Query across partitions with completely different access pattern (e.g., all customers who bought a specific product) |

**⚠️ EXAM TRAP:** LSI must be created at table creation. GSI can be added anytime. If question says you need a new query pattern on an **existing** table → answer is **GSI** (LSI not an option).

---

## 4. Capacity Modes — On-Demand vs Provisioned

### On-Demand Mode — Pay Per Request

| Attribute | Detail |
|-----------|--------|
| **Capacity planning** | None — instantly handles any traffic |
| **Cost** | ~$1.25 per million writes, ~$0.25 per million reads |
| **Zero traffic cost** | $0 |
| **Scaling** | Automatic — never throttles |
| **Best for** | New apps, unpredictable spiky workloads, infrequent use, unknown traffic patterns |
| **🇳🇬 Nigerian example** | Konga Yakata Sale (72-hour spike 500× then near-zero) — On-Demand absorbs spike automatically |

### Provisioned Mode — You Set Capacity

| Attribute | Detail |
|-----------|--------|
| **RCU** | 1 Read Capacity Unit = 1 strongly consistent 4KB read/sec (or 2 eventually consistent) |
| **WCU** | 1 Write Capacity Unit = 1 write up to 1KB/sec |
| **Cost** | Cheaper per request when traffic is steady and predictable |
| **Auto Scaling** | Set min/max — DynamoDB adjusts within range automatically |
| **Risk** | Throttling — if traffic exceeds provisioned capacity, requests are rejected |
| **Best for** | Predictable workloads, cost-optimized production with steady traffic |
| **🇳🇬 Nigerian example** | Bank HR portal (1,200 staff, 8am-5pm weekdays) — Provisioned + Auto Scaling saves 60% vs On-Demand |

### Capacity Units Explained

| Unit | Formula |
|------|---------|
| **RCU** | 1 RCU = 1 strongly consistent 4KB read/sec OR 2 eventually consistent 4KB reads/sec |
| **WCU** | 1 WCU = 1 write up to 1KB/sec (writes are always strongly consistent) |

---

## 5. Advanced DynamoDB Features

### DAX — DynamoDB Accelerator

| Attribute | Detail |
|-----------|--------|
| **What** | In-memory cache built exclusively for DynamoDB |
| **Speed** | Milliseconds → **Microseconds** (up to 10× faster reads) |
| **Integration** | Drop-in — replaces DynamoDB SDK calls, no app code changes |
| **Availability** | Multi-AZ, fully managed |
| **Use when** | Same data read repeatedly thousands of times/sec |
| **Skip when** | Write-heavy workload (DAX doesn't accelerate writes) |
| **Example** | Product page showing same top-10 trending items to thousands of visitors → DAX caches reads |

### DynamoDB Streams — Event-Driven Architecture

| Attribute | Detail |
|-----------|--------|
| **What** | Real-time log of every item change (create/update/delete) |
| **Retention** | 24-hour window of changes |
| **Trigger** | Automatically invokes Lambda when item changes |
| **Record contains** | BEFORE and AFTER state of changed item |
| **Use for** | Audit logs, notifications, cross-table sync, analytics |
| **Pattern** | DynamoDB change → Stream → Lambda → action |
| **Example** | Patient record updated → Stream triggers Lambda → Doctor's app gets push notification |

### Global Tables — Multi-Region Active-Active

| Attribute | Detail |
|-----------|--------|
| **What** | DynamoDB replicated across multiple AWS regions |
| **Access pattern** | Active-active — reads AND writes in every region (not just one) |
| **Replication speed** | Changes propagate globally in under 1 second |
| **Conflict resolution** | AWS automatically resolves write conflicts (last-writer-wins) |
| **Use when** | Global app needs low latency in multiple regions |
| **Cost** | You pay for storage and replication in every region |
| **Example** | Nigerian fintech with users in Lagos, London, New York — each region reads/writes to nearest replica |

---

## 6. Real-World Nigerian Scenario — PayFast Digital Payments App

**Context:** 8 million users daily. Diverse data workloads demand a purpose-built multi-table DynamoDB architecture.

### User Profiles Table

| Design | Detail |
|--------|--------|
| **Primary Key** | Simple PK: `UserID` |
| **Access pattern** | Low-volume reads (~1 read per login) |
| **Consistency** | Strongly consistent (users see latest balance/profile) |

### Transaction History Table

| Design | Detail |
|--------|--------|
| **Primary Key** | Composite: PK=`UserID`, SK=`TransactionTimestamp` |
| **GSI** | PK=`MerchantID`, SK=`Amount` |
| **Query patterns** | All transactions for user (sorted by date); all transactions for merchant (sorted by amount) |

### Active Sessions Table

| Design | Detail |
|--------|--------|
| **Primary Key** | Simple PK: `SessionToken` |
| **TTL** | Expiry timestamp attribute (e.g., 24 hours) |
| **Benefit** | DynamoDB's built-in TTL automatically deletes expired items — saves cost, no custom cleanup jobs |

### Notification Events — Streams + Lambda

| Design | Detail |
|--------|--------|
| **Streams** | Enabled on TransactionHistory table |
| **Trigger** | Lambda invoked on every change |
| **Process** | Lambda reads 'AFTER' image of changed item |
| **Result** | Push notification to user's phone in under 500ms — zero polling, fully event-driven |

**Discussion:** For a savings app like PiggyVest or Cowrywise, what tables would you design? What would the partition keys be?

---

## 7. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| **"LSI can be added to existing table"** | ❌ LSI must be created at **table creation**. For new query pattern on existing table → **GSI**. |
| **"Scan is efficient for finding items"** | ❌ **Scan** reads every item in table (slow, expensive). **Query** reads only items matching partition key (fast, cost-efficient). Never use Scan in production. |
| **"Hot partition is a DynamoDB bug"** | ❌ Hot partition = **poor partition key design** (low cardinality keys like Status, Date). Fix: high-cardinality keys (UserID, OrderID, UUID). |
| **"On-Demand is always cheaper"** | ❌ On-Demand costs **more per request** than Provisioned. For steady predictable workloads, Provisioned + Auto Scaling can be 70% cheaper. |

---

## 8. Practice Question (SAA-C03 Style)

**Scenario:** A media company stores articles in DynamoDB. They need to:
1. Retrieve all articles by a specific journalist sorted by publication date
2. Search all articles by news category regardless of journalist
3. Automatically delete articles after 90 days

**Which DynamoDB design BEST meets all three requirements?**

| Option | Answer |
|--------|--------|
| Simple PK on ArticleID; category filtering via Scan; scheduled Lambda for deletion | ❌ Scan is expensive/slow on large tables. Lambda adds complexity (TTL is free). |
| **Composite key (JournalistID + PubDate); GSI on CategoryName; TTL attribute set to 90 days** | ✅ **Correct** — composite key serves query 1. GSI serves query 2 (different PK). TTL serves query 3. |
| Composite key (JournalistID + PubDate); LSI on CategoryName; TTL attribute | ❌ LSI uses **same** partition key. CategoryName needs different PK → must be GSI. |
| Simple PK on JournalistID; Sort key on CategoryName; Lambda for auto-deletion | ❌ If SK is CategoryName, can't sort by date. Lambda adds complexity. |

**Answer:** B — Composite key for date-sorted journalist queries · GSI (not LSI) for cross-journalist category search · TTL for free auto-deletion.

---

## 9. Quick Reference Card

| Question | Answer |
|----------|--------|
| DynamoDB scaling method | Horizontal (automatic partitioning) |
| Primary key types | Simple (PK only) or Composite (PK + SK) |
| LSI creation timing | At table creation only (cannot add later) |
| GSI creation timing | Any time (can be added to existing table) |
| Max LSIs | 5 |
| Max GSIs | 20 |
| RCU = | 1 strongly consistent 4KB read/sec |
| WCU = | 1 write up to 1KB/sec |
| On-Demand best for | Unpredictable/spiky traffic |
| Provisioned + Auto Scaling best for | Steady, predictable workloads |
| DAX speed | Milliseconds → Microseconds (10× faster reads) |
| Streams retention | 24 hours |
| Global Tables replication lag | <1 second |
| TTL feature | Automatic item deletion (no Lambda needed) |

---

## 10. One Sentence to Remember

> **DynamoDB = serverless NoSQL, horizontal scaling, single-digit ms at any scale. Composite PK (PK+SK) enables range queries. LSI = same PK, new SK (table creation only). GSI = new PK (anytime). On-Demand for spiky traffic; Provisioned + Auto Scaling for steady workloads (70% cheaper). DAX = microsecond caching. Streams → Lambda = event-driven. Global Tables = multi-region active-active (<1 sec). TTL = auto-delete. Never use Scan in production.**
