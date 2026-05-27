# ☁️ Week 5 · Day 2: S3 Storage Classes & Lifecycle Rules

**Module:** Storage  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Not all data is created equal — and not all data should be stored the same way. S3 offers seven storage classes, each designed for a different access pattern, retrieval speed, and cost profile.

**The beginner mistake:** Store every file in S3 Standard forever. Old receipts, stale logs, and archived uploads accumulate silently, charging premium rates even when nobody touches them.

**The engineer mindset:** Before uploading, ask:
- How often will this file be opened?
- How fast must we retrieve it?
- How long must we keep it?
- Can we recreate it if it disappears?

---

## 2. The Mental Model — Data Temperature

| Temperature | Storage Class | Access pattern | Example |
|-------------|---------------|----------------|---------|
| 🔥 **Hot** | S3 Standard | Daily/weekly | Profile photos, active documents, current invoices |
| 🌤️ **Warm** | Standard-IA / One Zone-IA | Sometimes (fast when needed) | Old invoices, monthly reports, last quarter's exports |
| 🧊 **Cold** | Glacier Instant / Flexible | Rarely (compliance/audit) | Medical scans, audit files, compliance archives |
| ❄️ **Frozen** | Glacier Deep Archive | Almost never (years) | Legal records, regulatory retention, disaster recovery |

**Rule of thumb:** Colder data is cheaper to store, but retrieval becomes slower or costlier. Never sacrifice retrieval speed for active data just to save storage cost.

---

## 3. The 7 Storage Classes — Complete Reference

| Storage Class | Best for | Retrieval Speed | Min Duration | Watch Out For |
|---------------|----------|-----------------|--------------|----------------|
| **S3 Standard** | Frequent access | Milliseconds | None | Highest cost — use only for active data |
| **Intelligent-Tiering** | Unknown/changing patterns | Automatic tiers | None | Monthly monitoring fee per 1,000 objects |
| **Standard-IA** | Infrequent but fast access | Milliseconds | **30 days** | Retrieval fee + early deletion charge |
| **One Zone-IA** | Recreatable, infrequent data | Milliseconds | **30 days** | Single AZ only — data loss risk if AZ fails |
| **Glacier Instant** | Archive with fast retrieval | Milliseconds | **90 days** | Retrieval cost applies |
| **Glacier Flexible** | Rare archive, compliance | Minutes to hours | **90 days** | Must restore before accessing |
| **Glacier Deep Archive** | Long-term legal records | Hours (up to 12) | **180 days** | Slowest retrieval, highest early deletion penalty |

---

## 4. Fast Retrieval Classes (Millisecond Access)

### S3 Standard — $0.023/GB-month

| Attribute | Detail |
|-----------|--------|
| Best for | Active production data accessed daily or weekly |
| Retrieval fee | None |
| Minimum duration | None |
| Durability | Multi-AZ (11 nines) |
| Use for | `active/`, `users/`, `thumbnails/` prefixes |

### Standard-IA — $0.0125/GB-month

| Attribute | Detail |
|-----------|--------|
| Best for | Infrequent but urgent access (old invoices, monthly reports, backups) |
| Retrieval fee | **Yes** — per GB when accessed |
| Minimum duration | **30 days** — delete early, pay remaining days as penalty |
| Durability | Multi-AZ |
| Use for | Data rarely opened but must open fast when needed |

### One Zone-IA — $0.01/GB-month

| Attribute | Detail |
|-----------|--------|
| Best for | Recreatable data only — thumbnail caches, transcoded derivatives, temporary exports |
| Retrieval fee | Yes |
| Minimum duration | 30 days |
| Durability | **Single AZ only** — data loss risk if that zone fails |
| **Never use for** | Original user uploads, financial records, irreplaceable data |

---

## 5. Archive Classes (Must Understand Restore)

**Critical operational reality:** Glacier Flexible and Deep Archive objects must be **restored** before they can be accessed normally. You cannot simply read them like Standard objects.

### Glacier Instant Retrieval — $0.004/GB-month

| Attribute | Detail |
|-----------|--------|
| Best for | Archives that must open fast — compliance docs, audit files, medical records |
| Retrieval | **Milliseconds** — no restore job required |
| Minimum duration | 90 days |
| Retrieval cost | Yes (per GB) |

### Glacier Flexible Retrieval — $0.0036/GB-month

| Attribute | Detail |
|-----------|--------|
| Best for | Compliance archives, cold backups where speed isn't critical |
| Retrieval | **Minutes to hours** — must initiate restore job |
| Retrieval options | Expedited (1-5 min), Standard (3-5 hours), Bulk (5-12 hours) |
| Minimum duration | 90 days |

### Glacier Deep Archive — $0.00099/GB-month

| Attribute | Detail |
|-----------|--------|
| Best for | Long-term legal records, disaster recovery archives, 7-10 year retention |
| Retrieval | **Hours (up to 12)** — standard retrieval 12 hours, bulk up to 48 hours |
| Minimum duration | **180 days** |
| Cost savings | ~95% cheaper than S3 Standard |

**Warning:** Never promise users "instant access to all records" while storing data in Glacier Deep Archive. Align your SLA with actual retrieval capabilities.

---

## 6. Lifecycle Rules — Automate Cost Control

Manual storage class management does not scale. Lifecycle rules automate transitions, expiration, and cleanup.

### Lifecycle Actions

| Action | What it does |
|--------|--------------|
| **Transition** | Move objects to lower-cost storage class after X days |
| **Expiration** | Delete current objects after retention period |
| **Noncurrent version expiration** | Clean up old versions when versioning is enabled |
| **Incomplete multipart upload cleanup** | Remove abandoned uploads that charge storage |

### Typical Lifecycle Progression
Day 0: S3 Standard (active data, frequent access)
↓
Day 30: Standard-IA (infrequent access, fast retrieval)
↓
Day 90+: Glacier (archive storage, rare access)
↓
Year 7: Expire or Retain (compliance decision point)


### Design Guardrails

| Rule | Why |
|------|-----|
| Different prefixes need different policies | Don't apply one lifecycle rule to entire bucket |
| Glacier restore ≠ lifecycle transition | Different problems |
| Test in non-production bucket first | Prevent unintended data loss |
| Document lifecycle policy decisions | Team must understand cost/access implications |

---

## 7. Architecture Pattern — Prefixes + Lifecycle

Organize objects by prefix, attach lifecycle policies to each prefix.
bucket: basestack-prod-storage
│
├── active/ → S3 Standard (no transition)
│ └── Profile photos, current documents (accessed often)
│
├── receipts/ → Standard → Standard-IA after 30 days
│ └── Needed fast for disputes, not accessed daily
│
├── audit-logs/ → Standard → Glacier after 90 days
│ └── Compliance data, rarely accessed, must retain
│
└── temp/ → Expire after 7 days
└── Temporary exports, processing intermediates (never accumulate)


**Engineering rule:** Storage class decisions do not replace IAM policies, encryption, logging, or retention controls. Prefixes are organizational — layer security on top.

---

## 8. Real-World Scenario (Ibadan Health-Tech)

**Context:** Health-tech stores patient scans, consent forms, and diagnostic reports. New scans opened often in first 30 days. Older scans may be needed within 6 months for follow-up. Long-term retention required by NDPR compliance.

### Lifecycle Plan

| Time | Storage Class | Why |
|------|---------------|-----|
| **Day 0-30** | S3 Standard | Active care — doctors/nurses access often. Millisecond access, no retrieval fees. |
| **Day 30-180** | Standard-IA or Glacier Instant | Follow-up consultations. Fast retrieval preserved, cost reduced. |
| **Year 1+** | Glacier Flexible or Deep Archive | Long-term retention required by compliance. Initiate restore only when formally requested. |
| **Year 7** | Retain or Expire | Some records kept 7+ years by law. Temporary exports expired earlier. |

### Design Task Answers

| Question | Answer |
|----------|--------|
| Which file types can expire automatically? | Temporary exports, test uploads, processing intermediates |
| Security mistake to avoid | Storing PHI in One Zone-IA or without encryption |
| Sample lifecycle | Standard → Standard-IA/Glacier Instant → Glacier Flexible/Deep Archive |
| Temporary exports | Expire after 7 days |
| Consent forms | Retain 7+ years in Deep Archive |

---

## 9. Knowledge Check — Fintech Receipts

**Scenario:** Nigerian fintech stores transaction receipts accessed often for 30 days, sometimes for six months, retained for years for audit/compliance.

| ✅ Best Answer | ❌ Wrong Cheap Answer |
|----------------|----------------------|
| Standard → Standard-IA → Glacier/Deep Archive | One Zone-IA for critical receipts |
| Active receipts in Standard for 30 days | Sounds cost-effective but creates serious risk |
| Transition to Standard-IA for months 2-6 | Financial records cannot be recreated if AZ fails |
| Archive to Glacier Flexible or Deep Archive | Single-AZ storage never appropriate for irreplaceable financial data |

---

## 10. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "One Zone-IA is fine for critical data — it's cheaper" | ❌ **Single AZ only** — if that zone fails, data is lost. Never for irreplaceable data. |
| "Glacier Deep Archive objects can be accessed instantly" | ❌ Must **restore first** — takes hours (up to 12). Align SLA with reality. |
| "You can delete Glacier objects anytime with no penalty" | ❌ **Minimum duration charges** — 90 days for Glacier, 180 days for Deep Archive. |
| "Standard-IA has no retrieval fees" | ❌ **Retrieval fee per GB** when accessed. Factor into cost model. |
| "One lifecycle rule works for the entire bucket" | ❌ Different prefixes need **different policies**. Active vs archive vs temp. |
| "Intelligent-Tiering is always the best choice" | ❌ **Monitoring fee per 1,000 objects** — small objects may not benefit. |

---

## 11. Quick Reference Card

| Question | Answer |
|----------|--------|
| Cheapest storage class | Glacier Deep Archive ($0.00099/GB-month) |
| Fastest retrieval archive class | Glacier Instant (milliseconds) |
| Minimum duration for Standard-IA | 30 days |
| Minimum duration for Glacier Flexible | 90 days |
| Minimum duration for Deep Archive | 180 days |
| Single-AZ storage class | One Zone-IA only |
| No retrieval fee classes | S3 Standard only |
| Automates tier transitions | Intelligent-Tiering or Lifecycle rules |
| Must restore before accessing | Glacier Flexible and Deep Archive |
| Max cost savings vs Standard | ~95% (Deep Archive) |

---

## 12. One Sentence to Remember

> **Match storage class to access pattern: Hot = Standard (millisecond, no retrieval fee). Warm = Standard-IA (30-day min, retrieval fee). One Zone-IA = recreatable data only (single AZ risk). Cold = Glacier Instant (millisecond archive) or Flexible (minutes-hours, must restore). Frozen = Deep Archive (hours, 180-day min, ~95% cheaper). Lifecycle rules automate transitions by prefix. Never put irreplaceable data in One Zone-IA.**
