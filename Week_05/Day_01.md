# ☁️ Week 5 · Day 1: S3 Fundamentals

**Module:** Storage  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Amazon S3 (Simple Storage Service) is AWS's flagship object storage service. Launched in 2006, it stores more data than any other storage service on Earth. Netflix streams from S3. Spotify stores audio on S3. Every Nigerian fintech using AWS — Paystack, Flutterwave, Cowrywise — relies on S3 for logs, backups, user data, and compliance archives.

**The durability promise:** 11 nines — 99.999999999%. If you stored 10 million files in S3, you would expect to lose one file every 10,000 years. Achieved by automatically replicating every object across multiple Availability Zones within a Region.

**Why S3 appears on the exam:** Storage classes, bucket policies, replication, lifecycle policies, and pre-signed URLs are all heavily tested topics.

---

## 2. Object vs File Storage — The Mental Shift

| File storage (your laptop) | Object storage (S3) |
|---------------------------|---------------------|
| Files in folders inside folders | Everything is flat |
| Navigate hierarchical paths | Objects live in a bucket with a key |
| OS manages directories and file locks | No real folders — slash is display only |
| Not infinitely scalable | Infinitely scalable |

**Example:** `folder/subfolder/file.pdf` is just the key name. There is no real folder. This flat architecture makes S3 infinitely scalable without directory overhead.

---

## 3. Core Concepts — Buckets, Objects, Keys & Consistency

### S3 Bucket

| Attribute | Detail |
|-----------|--------|
| What it is | A container for objects |
| **Naming rule** | **Globally unique** across ALL AWS accounts worldwide |
| Region | Lives in a specific Region |
| Default limit | 100 buckets per account (can be increased) |

**Exam trap:** Bucket names are globally unique across ALL AWS accounts. If someone else owns `my-bucket`, you cannot create it — anywhere in the world.

### Object Key & Flat Namespace

| Attribute | Detail |
|-----------|--------|
| Key | The full "path" — e.g., `reports/2024/january.pdf` |
| Real folders? | **No** — slash is purely a display convention |
| Performance tip | For high throughput (thousands req/sec), use randomized prefixes (hash-based) to avoid hot partitions |

**Exam trap:** S3 uses a flat namespace. "Folders" in the console are just key prefixes.

### S3 Object

| Attribute | Detail |
|-----------|--------|
| Components | Key (name), Value (data), Metadata, Version ID, ETag (MD5 hash) |
| Max size | **5TB** per object |
| Max single PUT | 5GB (use Multipart Upload for larger) |
| Mutability | **Immutable** — cannot partially edit. Replace entire object to change. |

### S3 Consistency Model (Updated Dec 2020)

| Operation | Consistency |
|-----------|-------------|
| PUT → GET | **Strong** — returns new object immediately |
| DELETE → GET | **Strong** — returns 404 immediately |
| Overwrites | **Strong** — immediate consistency |

**Exam note:** Pre-2020 exam questions reference eventual consistency — ignore them. S3 is now **strongly consistent** for all operations.

---

## 4. S3 Storage Classes — Right-Size Your Cost

Core principle: match storage class to access pattern. Active data → Standard. Rarely accessed → Glacier.

| Storage Class | Storage Cost | Retrieval | Min Storage | Best for |
|---------------|--------------|-----------|-------------|----------|
| **S3 Standard** | $0.023/GB | Instant | None | Active data, web apps, content distribution |
| **Intelligent-Tiering** | $0.023/GB + monitoring fee | Instant | None | Unknown/changing access patterns |
| **Standard-IA** | $0.0125/GB + retrieval fee | Instant | **30 days** | Infrequent access, backups, DR copies |
| **One Zone-IA** | $0.01/GB + retrieval fee | Instant | **30 days** | Recreatable data, secondary backups |
| **Glacier Instant** | $0.004/GB | Milliseconds | **90 days** | Archives needing occasional fast access |
| **Glacier Flexible** | $0.0036/GB | 1 min–12 hrs | **90 days** | Backup archives, quarterly/yearly access |
| **Glacier Deep Archive** | $0.00099/GB | 12–48 hrs | **180 days** | Regulatory archives (7-10 year retention) |

### Minimum Duration Charges (Exam Critical)

| Class | Minimum storage duration |
|-------|-------------------------|
| Standard-IA / One Zone-IA | 30 days |
| Glacier Instant / Flexible | 90 days |
| Glacier Deep Archive | 180 days |

**Delete early → you still pay for the full minimum period.**

### Intelligent-Tiering Advantage

- Only class with **no retrieval fees** and **no minimum storage duration**
- Automatically moves objects between tiers based on access patterns
- Small monthly monitoring fee per object

### One Zone-IA Risk

Stored in a **single AZ** — not replicated across AZs. Data loss risk if that AZ fails. Only use for data that can be recreated.

---

## 5. S3 Security — Bucket Policies, Encryption & Pre-Signed URLs

S3 is the most commonly misconfigured AWS service. A single misconfigured bucket policy can expose sensitive data to the entire internet.

### Bucket Policies

JSON-based resource policies applied directly to the bucket.

| Element | What it defines |
|---------|-----------------|
| **Principal** | Who (which user, account, or service) |
| **Action** | What (e.g., `s3:GetObject`, `s3:PutObject`) |
| **Resource** | Which bucket/object |
| **Condition** | When/under what circumstances (e.g., IP address, MFA) |

### Block Public Access

Four settings that block all forms of public access — **even if a bucket policy tries to grant public access**.

| Setting | What it does |
|---------|--------------|
| Block public ACLs | Prevents public access via ACLs |
| Block public bucket policies | Prevents public access via bucket policies |
| Ignore public ACLs | Ignores any public ACLs |
| Restrict public buckets | Additional protection for buckets with public policies |

**For a public website bucket:** You MUST disable Block Public Access AND add a bucket policy allowing `s3:GetObject` from `*`. Both conditions must be true.

**Exam trap:** Block Public Access OVERRIDES bucket policies. If BPA is enabled, public bucket policies are blocked.

### S3 Encryption Options

| Type | How it works | Use when |
|------|--------------|----------|
| **SSE-S3** | AWS manages keys (AES-256). No extra cost. Default. | "Encrypt data at rest" — no specific key requirements |
| **SSE-KMS** | AWS KMS customer-managed keys. Audit trail via CloudTrail. Adds KMS costs. | Need audit logs, key rotation control, customer-managed keys |
| **SSE-C** | You provide the key. AWS encrypts/decrypts but never stores key. HTTPS required. | "Bring your own key" without KMS |
| **Client-Side** | You encrypt before upload. AWS never sees plaintext. | Maximum security, maximum complexity |

**Exam clue:** SSE-KMS = "encryption with audit trail" or "customer-managed keys." SSE-S3 = "encrypt with AWS-managed keys."

### Pre-Signed URLs

Generate a time-limited URL that grants temporary access to a private S3 object — without making the bucket public.

| Attribute | Detail |
|-----------|--------|
| How it works | URL contains authentication credentials signed with your AWS credentials |
| Expiry | 1 second to **7 days** (max) |
| After expiry | Returns 403 Access Denied |
| Use cases | User downloads invoice (15 min). Browser uploads directly to S3. Share report with external auditor (24 hours). |

**Exam trap:** Pre-signed URLs are the answer for "temporary access to private S3 objects without changing bucket permissions."

---

## 6. Data Management — Versioning, Lifecycle & Replication

### S3 Versioning

| Attribute | Detail |
|-----------|--------|
| What it does | Keeps **all versions** of every object |
| Delete behavior | Adds a **Delete Marker** — object appears deleted, but versions still exist |
| Restore | Delete the Delete Marker |
| Cannot fully disable | Can be suspended, but not fully disabled |
| Cost | You pay for storage of ALL versions |

**MFA Delete:** Requires MFA to permanently delete versions or disable versioning. Protects against accidental or malicious deletion.

**Exam trap:** Deleting an object in a versioned bucket does NOT permanently delete it. It adds a Delete Marker. The object still exists and is still billed.

### Lifecycle Policies

Automate how objects change over time — transition or expire.

| Action | What it does | Example |
|--------|--------------|---------|
| **Transition** | Move to cheaper storage class after X days | Standard → Standard-IA after 30 days |
| **Expiration** | Permanently delete after X days | Delete logs after 90 days |

**Exam tip:** Automatically moving data to lower-cost storage over time = lifecycle policy. Recovering from deletion/overwrite = versioning.

### Replication

Copy objects from one bucket to another automatically.

| Type | What it does | Use case |
|------|--------------|----------|
| **SRR (Same-Region)** | Copies within same region | Operational isolation, separate accounts |
| **CRR (Cross-Region)** | Copies to another region | Disaster recovery, geo redundancy, lower latency in another region |

**Requirements:** Versioning must be enabled on **both** source and destination buckets.

**Exam clue:** "Disaster recovery" or "copying to another region" = Cross-Region Replication.

### How They Work Together (Production Pattern)
Versioning (protects from deletion/overwrite)
+
Lifecycle (manages old versions, transitions cold data)
+
Replication (DR copy in another region)


---

## 7. S3 Pricing Components

| Component | S3 Standard | Glacier Deep Archive | Notes |
|-----------|-------------|---------------------|-------|
| **Storage** | $0.023/GB/month | $0.00099/GB/month | Per GB stored |
| **PUT/COPY/POST/LIST** | $0.005/1,000 requests | $0.05/1,000 requests | Every upload costs |
| **GET/SELECT** | $0.0004/1,000 requests | $0.0004/1,000 requests | Every download costs |
| **Data Transfer IN** | **FREE** | **FREE** | Uploading is always free |
| **Data Transfer OUT (internet)** | $0.09/GB (first 10TB) | Same | Use CloudFront to reduce |
| **Retrieval Fee** | None | Per GB retrieved | Factor into cost models |

---

## 8. Multipart Upload

For objects **larger than 5GB** (up to 5TB).

| Benefit | Detail |
|---------|--------|
| Speed | Upload parts in parallel |
| Resilience | Resume after failure — only retry failed parts |
| Max parts | 10,000 parts (minimum 5MB each) |

**Exam trap:** Incomplete multipart uploads accrue storage charges. Use a Lifecycle rule to abort incomplete uploads after 7 days to avoid surprise costs.

---

## 9. S3 Object Lock — WORM Storage

Write Once, Read Many — objects cannot be deleted or overwritten for a defined retention period.

| Mode | Who can delete | Use case |
|------|----------------|----------|
| **Governance mode** | Users with special permissions | Internal compliance |
| **Compliance mode** | **NO ONE** — not even root | FINRA, SEC, CBN, FIRS record-keeping |

**Retention types:**
- **Retention period** — fixed date (e.g., keep until Dec 31, 2030)
- **Legal Hold** — indefinite until removed (overrides retention period)

**Exam trap:** Compliance mode is the only S3 feature that provides truly immutable, undeletable storage — not even root can delete.

---

## 10. Real-World Scenario (Legal Document Platform)

**Context:** Document platform for Nigerian law firms. Legal documents (PDFs, contracts, court filings) stored in S3. Must satisfy CBN and FIRS record-keeping requirements of 7 years while enabling secure, time-limited document sharing with clients.

### Solution Components

| Feature | Implementation | Why |
|---------|----------------|-----|
| **Pre-signed URLs** | 1-hour expiry for client document access | Bucket stays private. Client gets single document for limited time. |
| **Lifecycle Policy** | Standard (0-30 days) → Standard-IA (30-90 days) → Glacier Flexible (90 days-7 years) → Glacier Deep Archive (7-10 years) → Delete (10+ years) | Fully automated. No manual intervention. No forgotten archives. |
| **CRR (Cross-Region Replication)** | Replicate from `af-south-1` (Cape Town) to `eu-west-1` (Ireland) | Disaster recovery. If Cape Town has outage, access documents from Ireland. |

---

## 11. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Bucket names must be unique within your account" | ❌ **Globally unique** across ALL AWS accounts worldwide |
| "S3 has folders and directories like a file system" | ❌ **Flat namespace** — slashes are display conventions only |
| "Public access requires only a bucket policy" | ❌ Need **BOTH**: disable Block Public Access AND bucket policy granting `*` |
| "Deleting an object in versioned bucket permanently removes it" | ❌ Adds a **Delete Marker** — object still exists and is still billed |
| "You can delete Glacier objects anytime with no penalty" | ❌ **Minimum duration charges** — 30/90/180 days depending on class |
| "Compliance mode can be overridden by root" | ❌ **No one** can delete — not even root |

---

## 12. Practice Question (SAA-C03 Style)

**Scenario:** Legal tech company stores confidential client documents in S3. Documents must be retained for 7 years with **no possibility of deletion** — even by administrators. Lawyer occasionally needs to share a specific document with a client for a limited time without exposing the bucket publicly.

**Which TWO features meet these requirements?**

| Option | Answer |
|--------|--------|
| Enable S3 Versioning | ❌ Admin can still permanently delete all versions |
| **Enable S3 Object Lock in Compliance Mode** | ✅ **Correct** — prevents ANYONE (including root) from deleting before retention expires |
| Make bucket public + Security Groups | ❌ S3 doesn't support Security Groups. Public bucket exposes ALL documents. |
| **Generate Pre-Signed URLs with Expiry** | ✅ **Correct** — temporary, time-limited access to specific private objects |

---

## 13. Quick Reference Card

| Question | Answer |
|----------|--------|
| Bucket name scope | Globally unique across ALL AWS accounts |
| Max object size | 5TB |
| Max single PUT | 5GB (use Multipart Upload for larger) |
| S3 consistency model | Strong (since Dec 2020) |
| Default encryption | SSE-S3 (AES-256, free) |
| Encryption with audit trail | SSE-KMS |
| Pre-signed URL max expiry | 7 days |
| Versioning delete behavior | Adds Delete Marker (not permanent) |
| MFA Delete requires | MFA to permanently delete or disable versioning |
| Lifecycle actions | Transition (move) or Expiration (delete) |
| Replication requirement | Versioning on **both** buckets |
| Immutable storage (no root delete) | Object Lock — Compliance mode |

---

## 14. One Sentence to Remember

> **S3 = globally unique bucket names, flat namespace (no real folders), strongly consistent. Match storage class to access pattern — Standard for active, Glacier for archives. Block Public Access overrides bucket policies. Pre-signed URLs for temporary access. Versioning keeps all versions (Delete Marker, not permanent). Lifecycle automates tier transitions. CRR for disaster recovery. Object Lock Compliance mode = no one can delete — not even root.**
