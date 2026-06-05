# ☁️ Week 5 · Day 5: Hybrid Storage — Storage Gateway & DataSync

**Module:** Storage  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why Hybrid Storage Matters

Most established enterprises run on legacy on-premises infrastructure — physical file servers, NAS devices, and physical tape libraries. Moving 100% of these workloads to the cloud overnight is impractical due to:

- Massive data volumes
- Legacy applications hardcoded for local file shares
- Strict regulatory requirements mandating local data copies

**AWS Hybrid Storage** bridges this gap by acting as a local extension of your AWS VPC, enabling cloud benefits without disrupting ongoing business operations.

### Business Benefits

| Benefit | How |
|---------|-----|
| **Cost savings** | Replace capital-heavy physical NAS with pay-as-you-go S3 |
| **Business continuity** | Hybrid disaster recovery backups |
| **Compliance ready** | Keep local copies while archiving to cloud |

---

## 2. Core Concepts

### AWS Storage Gateway

A suite of hybrid cloud storage services connecting on-premises applications to AWS storage. Runs as a **virtual machine appliance** (VMware ESXi, Hyper-V, KVM) in your local data center.

### AWS DataSync

An online data transfer service that automates and accelerates moving massive datasets between on-premises storage and AWS. Operates **up to 10x faster** than standard open-source tools.

### The Four Gateway Types

| Gateway | Protocol | Backend | Best for |
|---------|----------|---------|----------|
| **S3 File Gateway** | NFS / SMB | Amazon S3 | Branch office files, home directories, web content |
| **FSx File Gateway** | SMB | Amazon FSx | Windows workloads needing NTFS ACLs, shadow copies, DFS |
| **Volume Gateway** | iSCSI | S3 + EBS | Block volumes (cached or stored mode) |
| **Tape Gateway** | iSCSI VTL | S3 + Glacier | Replace physical tape libraries |

---

## 3. Deep Dive — The Four Gateway Types

### S3 File Gateway

| Attribute | Detail |
|-----------|--------|
| **Protocol** | NFS (Linux) or SMB (Windows) |
| **Backend** | Amazon S3 |
| **How it works** | Local files become **native S3 objects** — 1:1 mapping |
| **Key advantage** | Can apply S3 Lifecycle policies, Versioning, Object Lock directly |
| **Best for** | Branch office files, home directories, web content repositories |

### FSx File Gateway

| Attribute | Detail |
|-----------|--------|
| **Protocol** | SMB only |
| **Backend** | Amazon FSx for Windows File Server |
| **Key advantage** | Full NTFS ACLs, shadow copies, DFS Namespace support |
| **Best for** | Windows workloads requiring native Windows file server features |

### Volume Gateway — Two Modes (Exam Critical)

| Mode | Primary copy location | Local cache | Best for |
|------|----------------------|-------------|----------|
| **Cached Volume** | Amazon S3 | Frequently accessed (hot) data | Capacity-constrained on-premises storage |
| **Stored Volume** | On-premises | Full dataset | Low-latency requirements, limited cloud bandwidth |

**Exam distinction:**
- **Cached** = primary data in S3, cache local
- **Stored** = primary data local, backups (snapshots) to S3

### Tape Gateway

| Attribute | Detail |
|-----------|--------|
| **Protocol** | iSCSI Virtual Tape Library (VTL) |
| **Backend** | S3 + **Glacier** (not S3 Standard) |
| **What it does** | Eliminates physical tape infrastructure — mimics virtual tape library |
| **Archive target** | Amazon S3 Glacier Flexible Retrieval or Glacier Deep Archive |

**Exam trap:** Tape Gateway archives directly to **Glacier**, never S3 Standard.

---

## 4. AWS DataSync — How It Works

### Process Flow
Deploy Agent (lightweight VM on-premises)
↓

Create Task (define source and destination)
↓

Execute Transfer (compressed, encrypted, delta-only)
↓

Monitor & Schedule (recurring or one-time)


### Destination Targets Supported

| Target | Supported? |
|--------|------------|
| Amazon S3 (all tiers, including Glacier) | ✅ |
| Amazon EFS | ✅ |
| Amazon FSx (Windows, Lustre, NetApp ONTAP) | ✅ |

### Performance Metrics

| Metric | Value |
|--------|-------|
| Speed | Up to 10 Gbps per transfer task |
| Features | Built-in throttling, automatic recovery, delta sync |
| On-premises requirement | Agent VM (no agent needed for AWS-to-AWS transfers) |

---

## 5. Storage Gateway vs DataSync — Decision Matrix (Exam Critical)

| Factor | Storage Gateway | DataSync |
|--------|-----------------|----------|
| **Primary purpose** | Extend on-premises storage to cloud continuously | Fast, scheduled bulk copy or migration |
| **How applications see it** | Normal file share (NFS/SMB) or hard drive (iSCSI) | Background service — no client mount |
| **Data transfer** | Silent background upload as apps write | Bulk transfer of large data folders |
| **Latency** | Local cache provides sub-millisecond access | N/A (migration tool) |
| **Code changes required** | None | None |
| **Exam trigger words** | "Extend", "low-latency access", "continuous" | "Migrate", "transfer", "sync", "bulk copy" |

**Exam shortcut:**
- **Storage Gateway** = ongoing, continuous access with low latency
- **DataSync** = one-time migration or scheduled bulk sync

---

## 6. Real-World Case Study — TrustCo Microfinance, Abuja

### The Challenge

- 30 TB of legacy customer KYC files on aging NAS servers
- Local tellers require daily Windows SMB access
- CBN mandate: financial records stored immutably for **7 years**
- Scaling local infrastructure = unsustainable

### The Solution

| Step | Action | Why |
|------|--------|-----|
| 1 | **S3 File Gateway** deployed on-premises | Staff continue accessing SMB shares unaware data now streams to S3 |
| 2 | **DataSync** one-time migration of 30 TB legacy files | Secure copy to AWS in just 3 days |
| 3 | **S3 Object Lock** with 7-year retention | WORM compliance for regulatory audits |
| 4 | **Glacier Lifecycle** transition | Move old files to S3 Glacier — immediate cost savings |

**Result:** Continuous local access + cloud durability + compliance + cost savings.

---

## 7. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "File Gateway stores files as EBS volumes" | ❌ Files map **1:1 to S3 objects**. Not EBS, not EFS. |
| "Cached and Stored Volume Gateway are the same" | ❌ **Cached** = primary in S3 (cache local). **Stored** = primary local (snapshots to S3). |
| "DataSync is for continuous application access" | ❌ DataSync is for **migration/sync**. Storage Gateway is for continuous access with low latency. |
| "Tape Gateway archives to S3 Standard" | ❌ Archives directly to **S3 Glacier** (Flexible or Deep Archive). Never S3 Standard. |
| "FSx File Gateway uses NFS protocol" | ❌ **SMB only** — designed for Windows workloads with NTFS ACLs. |
| "Volume Gateway stores primary data in EBS" | ❌ Backend is **S3 + EBS snapshots**. Cached mode stores primary in S3. |

---

## 8. Practice Question (SAA-C03 Style)

**Scenario:** A Lagos bank has 40 TB of financial records on on-premises NAS servers. Loan officers access these files daily via Windows shared drives (SMB). The bank wants to reduce storage costs by moving data to Amazon S3 but **cannot disrupt daily file access workflow** of local staff.

**Which solution should a Solutions Architect recommend?**

| Option | Answer |
|--------|--------|
| Migrate all files to Amazon EFS and mount shares over Direct Connect | ❌ Disrupts workflow. Requires application changes. |
| **Deploy S3 File Gateway on-premises, configure SMB file shares backed by Amazon S3** | ✅ **Correct** — seamless local SMB access, apps unchanged, data goes to S3 |
| Use AWS DataSync to continuously sync files to S3 as applications write them | ❌ DataSync is for migration/sync, not continuous interactive access |
| Generate S3 presigned URLs for every file and update local workstation configurations | ❌ Impractical for 40 TB. Disrupts workflow. |

**Why B is correct:** S3 File Gateway provides seamless local SMB access transparent to Windows clients. Option C is incorrect because DataSync is not designed for continuous, interactive client access.

---

## 9. Quick Reference Card

| Question | Answer |
|----------|--------|
| S3 File Gateway protocols | NFS (Linux) + SMB (Windows) |
| FSx File Gateway protocol | SMB only |
| Volume Gateway protocols | iSCSI |
| Tape Gateway backend | S3 Glacier (not S3 Standard) |
| Cached Volume — primary copy location | Amazon S3 |
| Stored Volume — primary copy location | On-premises |
| DataSync max speed | Up to 10 Gbps per task |
| DataSync destination targets | S3, EFS, FSx |
| Storage Gateway exam trigger | "Extend", "continuous access", "low latency" |
| DataSync exam trigger | "Migrate", "bulk transfer", "sync" |

---

## 10. One Sentence to Remember

> **Storage Gateway extends on-premises storage to the cloud with low-latency local access (File = NFS/SMB to S3, Volume = iSCSI to S3/EBS, Tape = VTL to Glacier). DataSync is for high-speed bulk migrations (up to 10 Gbps). Cached Volume = primary in S3. Stored Volume = primary local. Tape Gateway archives to Glacier, never S3 Standard.**
