
# ☁️ Week 6 · Day 5: ElastiCache & In-Memory Databases

**Module:** Databases  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why In-Memory?

ElastiCache sits between your application and your database — storing frequently accessed data in RAM so that most requests are answered in under 1 millisecond.

### The Speed Gap

| Storage Type | Latency |
|--------------|---------|
| Hard Disk (HDD) | ~10 ms per read |
| Database Query (disk + network + compute) | 1–100 ms |
| **RAM / ElastiCache** | **< 1 ms** |

**The problem:** If your app gets 10,000 users at once and each triggers a database query, your database CPU spikes, connection limits hit, and response times degrade.

**The solution:** ElastiCache stores popular data in RAM — most requests never hit the database at all.

---

## 2. The Two ElastiCache Engines

### Redis — The Powerful, Feature-Rich Engine

| Feature | Support |
|---------|---------|
| Data structures | Lists, sets, sorted sets, hashes, strings |
| Persistence | ✅ RDB snapshots + AOF (data survives restarts) |
| Multi-AZ replication | ✅ High availability |
| Pub/Sub messaging | ✅ Real-time notifications |
| Encryption | ✅ At rest + in transit |
| Best for | Leaderboards, sessions, real-time data, complex caching |

### Memcached — The Simple, Fast Engine

| Feature | Support |
|---------|---------|
| Data structures | Simple key-value only |
| Persistence | ❌ Data lost on restart (RAM only) |
| Replication | ❌ No replication, no failover |
| Scaling | Multi-threaded — scales horizontally very well |
| Encryption | ❌ Not supported |
| Best for | Basic HTML caching, simple object lookups |

**SAA-C03 Exam Tip:** If question mentions **persistence, replication, or data structures** → answer is **Redis**. Simple cache only → **Memcached**.

---

## 3. Caching Patterns — How It Works

### Cache-Aside (Lazy Loading) — Most Common Pattern

| Step | Action |
|------|--------|
| 1 | **App checks cache** — Is data already in ElastiCache? |
| 2 | **Cache HIT** ✅ — Return cached data instantly. No database involved. |
| 3 | **Cache MISS** ❌ — Query database, get result |
| 4 | **Store + Return** — Save DB result to cache for next time. Return data to user |

**Use when:** Reads outnumber writes and you can tolerate slightly stale data.

### Write-Through — Keep Cache Always Fresh

| Step | Action |
|------|--------|
| 1 | **App writes data** (user updates profile, makes payment) |
| 2 | **Write to DB** — Data saved to main database |
| 3 | **Write to Cache** — Same data immediately written to ElastiCache |
| 4 | **Future reads** — Cache always fresh, no staleness |

**Use when:** Data accuracy is critical — banking, stock prices, user settings.

---

## 4. Architecture — Request Flow

**Normal flow (Cache HIT):** Request → App → ElastiCache → User — sub-1ms response.

**Fallback flow (Cache MISS):** Request → App → ElastiCache (miss) → DB → ElastiCache (store) → User — slower, but only once per key.

---

## 5. Redis Deep Dive — The Powerful Engine

| Feature | What it does |
|---------|--------------|
| **Data Structures** | Strings, lists, sets, sorted sets, hashes — enables leaderboards (sorted sets) and real-time analytics |
| **Persistence** | Snapshots (RDB) or append-only log (AOF) ensure data survives restarts |
| **Replication** | One primary + multiple read replicas for high throughput |
| **Multi-AZ Failover** | If primary fails, replica automatically promoted — zero downtime |
| **Pub/Sub Messaging** | Real-time notifications — chat apps, live feeds, event-driven architectures |
| **TTL (Expiry)** | Set expiry time on any key — cache auto-cleans itself |

**Real-world use:** Redis sorted sets power game leaderboards (top 10 scores) in microseconds — try doing that with SQL on millions of rows.

---

## 6. Real-World Nigerian Scenario — News Platform (Pulse.ng / Nairametrics)

**Problem:** "Top 10 Stories Today" panel on every page. 500,000 daily visitors. Each page load triggers heavy SQL query — counting views, sorting by date, filtering by category.

### Without ElastiCache

| Metric | Value |
|--------|-------|
| Database queries | 500,000× per day |
| Database CPU | 95% (constant alarms) |
| Page load time | 3,000–4,000 ms (3–4 seconds) |
| Monthly server bill | ₦850,000 |
| User behavior | Abandon site (slow loading) |

### With ElastiCache Redis

| Metric | Value |
|--------|-------|
| Top 10 stories computed | Once per minute |
| Cached result serves | All 500,000 users |
| Database load reduction | **95%** |
| Page load time | **8–12 ms (300× faster!)** |
| Monthly server bill | ₦90,000 |
| User behavior | Stay, engage, return |

### Results

| Improvement | Value |
|-------------|-------|
| Faster pages | **300×** (3–4 sec → 8–12 ms) |
| DB load reduction | **95%** |
| Monthly savings | **₦760,000** |

---

## 7. Decision Guide — Redis vs Memcached

| Requirement / Scenario | Redis | Memcached |
|------------------------|-------|-----------|
| Data survives restarts (persistence) | ✅ YES | ❌ NO |
| Complex data: lists, sets, leaderboards | ✅ YES | ❌ NO |
| Multi-AZ failover / replication | ✅ YES | ❌ NO |
| Real-time messaging (Pub/Sub) | ✅ YES | ❌ NO |
| Simple HTML / object caching | ✅ YES | ✅ YES |
| Maximum horizontal scaling with threads | ⚠ Limited | ✅ YES |
| Encryption at rest + in transit | ✅ YES | ❌ NO |
| Session storage for stateless apps | ✅ YES | ⚠ Possible |

**Choose Redis when:** You need persistence, complex data types, replication, Pub/Sub, or session management.

**Choose Memcached when:** You need maximum horizontal scaling for simple key-value lookups and don't need Redis advanced features.

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| **"Memcached is persistent"** | ❌ Memcached stores data **only in RAM**. If cluster restarts, all cached data is gone. Exam keyword: "data must survive restarts" → **Redis**. |
| **"ElastiCache is a primary database"** | ❌ ElastiCache is a **caching layer** — it does not replace RDS/DynamoDB. Exam keyword: "primary data store" → RDS/DynamoDB, not ElastiCache. |
| **"Cache invalidation is automatic"** | ❌ When you update data in DB, you must manually update/delete cache entry. **TTL** prevents stale data from living forever (short TTL = fresh data; long TTL = faster but risk stale). |
| **"Browser cookies work for session storage across servers"** | ❌ Cookies don't scale across multiple servers. Exam keyword: "stateless application" / "session management" → **ElastiCache Redis**. |

---

## 9. Practice Question (SAA-C03 Style)

**Scenario:** A Lagos fintech company stores user session tokens for their payment app on each web server. When they add a second server for scale, users get logged out randomly.

**Which AWS service fixes this?**

| Option | Answer |
|--------|--------|
| Amazon RDS Multi-AZ — replicate the session database | ❌ Would work but adds database load; overkill for session storage |
| **ElastiCache Redis — centralize session storage in memory** | ✅ **Correct** — Redis provides centralized, in-memory store accessible by all servers, enabling stateless architecture |
| Amazon S3 — store session files in a shared bucket | ❌ S3 is object storage — high latency for session lookups |
| Amazon CloudFront — cache sessions at the edge | ❌ CloudFront is CDN — caches static content, not dynamic sessions |

**Answer:** B — Redis provides a centralized, in-memory store accessible by all servers, enabling stateless architecture and solving the session sharing problem.

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| ElastiCache response time | <1 ms |
| Redis persistence | ✅ RDB snapshots + AOF |
| Memcached persistence | ❌ Data lost on restart |
| Redis data structures | Lists, sets, sorted sets, hashes, strings |
| Memcached data structures | Simple key-value only |
| Redis Multi-AZ | ✅ Automatic failover |
| Memcached replication | ❌ None |
| Cache-Aside pattern | Check cache first, fall back to DB on miss |
| Write-Through pattern | Update cache + DB together |
| TTL purpose | Prevents stale data from living forever |
| Session storage for stateless apps | ElastiCache Redis |
| Exam keyword "data must survive restarts" | → Redis (not Memcached) |
| Exam keyword "primary data store" | → RDS/DynamoDB (not ElastiCache) |

---

## 11. One Sentence to Remember

> **ElastiCache serves data from RAM (<1ms) — Redis for persistence, complex data types (lists, sets, sorted sets), Multi-AZ, Pub/Sub, and session storage. Memcached for simple key-value caching with horizontal scaling (no persistence). Cache-Aside = check cache first, DB on miss. Write-Through = update cache + DB together. TTL prevents stale data. ElastiCache is a cache, NOT a primary database.**
