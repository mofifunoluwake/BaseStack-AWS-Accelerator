# ☁️ Week 5 · Day 4: Amazon EFS & Amazon FSx — Shared File Storage

**Module:** Storage  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why Shared Storage Matters

While EBS is excellent for single-instance persistence, modern scalable applications require **concurrent access** to a shared file system.

### The EBS Problem

| Limitation | Why it matters |
|------------|----------------|
| **1-to-1 attachment** | Standard EBS attaches to one EC2 instance at a time (Multi-Attach io2 exists but is limited to same AZ) |
| **AZ-locked** | Volume in eu-west-1a cannot mount to instance in eu-west-1b |
| **No concurrent access** | Multiple instances cannot read/write same EBS volume without complex cluster software |

### The Solution — Managed File Storage

| Service | Protocol | OS | Use case |
|---------|----------|-----|----------|
| **Amazon EFS** | NFSv4 | Linux only | Cloud-native shared file storage across AZs |
| **FSx for Windows** | SMB | Windows + Linux | Native Windows file shares with AD integration |
| **FSx for Lustre** | Parallel FS | Linux | HPC, ML training, genomics, rendering |
| **FSx for NetApp ONTAP** | NFS/SMB/iSCSI | Any | Multi-protocol, instant snapshots, deduplication |
| **FSx for OpenZFS** | NFS | Linux | High throughput, extreme performance |

---

## 2. Amazon EFS — Fully Managed NFS for Linux

EFS provides serverless, highly available file storage. It handles growth automatically — petabytes of data with no manual capacity planning.

### Core Features

| Feature | Detail |
|---------|--------|
| **Protocol** | NFSv4.1/v4.0 — standard Linux utilities |
| **OS support** | **Linux only** (not natively for Windows) |
| **Resilience** | Multi-AZ by default — mount targets in each subnet |
| **Security** | Security Groups + IAM + KMS encryption at rest + TLS in transit |
| **Scaling** | Automatic — no provisioning needed |

### EFS One Zone — Cheaper Option

| Attribute | Regional EFS | One Zone EFS |
|-----------|--------------|--------------|
| **Storage cost** | Baseline | **47% cheaper** |
| **Resilience** | Multi-AZ | Single AZ only |
| **Best for** | Production | Dev/test, staging, recreatable data |

**Exam tip:** EFS is exclusively for Linux. If you see a Windows workload requiring shared storage, look for FSx in the options.

---

## 3. EFS Performance & Throughput

### Performance Modes (Choose at creation — cannot change)

| Mode | IOPS | Latency | Best for |
|------|------|---------|----------|
| **General Purpose (default)** | Up to 35,000 IOPS | Lowest latency | Web servers, CMS, developer home directories |
| **Max I/O** | Unlimited IOPS | Slightly higher metadata latency | Big data, massive parallel clusters |

### Throughput Modes (Can change dynamically)

| Mode | How it works | Best for |
|------|--------------|----------|
| **Bursting (default)** | Throughput scales with storage size (50 KB/s per GB) | Small to moderate workloads |
| **Provisioned** | You define MB/s regardless of size | Small repositories needing high speed |
| **Elastic (recommended)** | Automatically scales based on real-time demand | Spiky or unpredictable workloads |

### Lifecycle Tiering — Automatic Cost Optimization

Files not accessed for a defined threshold (7, 14, 30, 60, or 90 days) are automatically moved to **Infrequent Access (IA)** tier.

| Tier | Cost |
|------|------|
| Standard | ~$0.30/GB |
| IA | ~$0.025/GB |

**Savings:** ~90% for cold data.

---

## 4. Amazon FSx Family — Four Managed File Systems

| Service | Protocol | OS | Key feature |
|---------|----------|-----|-------------|
| **FSx for Windows** | SMB | Windows + Linux | Active Directory integration, native NTFS permissions |
| **FSx for Lustre** | Parallel FS | Linux | Sub-millisecond latency, millions IOPS, direct S3 integration |
| **FSx for NetApp ONTAP** | NFS/SMB/iSCSI | Any | Multi-protocol, instant snapshots, deduplication |
| **FSx for OpenZFS** | NFS | Linux | High throughput, extreme performance |

---

## 5. FSx for Windows vs FSx for Lustre (Exam Critical)

| Feature | FSx for Windows | FSx for Lustre |
|---------|-----------------|----------------|
| **Primary protocol** | SMB (v2.0–v3.1.1) | POSIX-compliant parallel file client |
| **Authentication** | Microsoft Active Directory | POSIX permissions + IAM |
| **OS compatibility** | Windows Server, Windows client, Linux (via SMB) | Linux only |
| **Deployment modes** | Single-AZ (dev/test) or Multi-AZ (active/passive failover) | Scratch (no replication, cheap) or Persistent (HA replica) |
| **S3 integration** | Manual copy or DataSync | **Direct bi-directional sync** (lazy loads, writes back) |
| **Use cases** | User directories, MS SQL Server, enterprise IIS web apps | HPC, ML training, genomics, rendering |

### FSx for Lustre — Scratch vs Persistent

| Mode | Replication | Data durability | Best for |
|------|-------------|-----------------|----------|
| **Scratch** | No replication (cheapest) | Data lost if host fails | Short-term processing, temporary workloads |
| **Persistent** | High availability replica | Data survives host failure | Long-term production HPC, ML training |

**Exam trap:** Long-term jobs require **Persistent** deployment. Scratch is for temporary processing only.

---

## 6. Architectural Decision Framework

Walk through these questions to pick the right service:

| Question | Answer leads to |
|----------|-----------------|
| Do you need shared access across multiple EC2 instances? | Yes → Continue. No → Use EBS. |
| Are your clients **Windows** or require SMB protocol? | **FSx for Windows File Server** |
| Are your clients **Linux** and need high-performance parallel processing? | **FSx for Lustre** |
| Are your clients **Linux** and need simple shared NFS across AZs? | **Amazon EFS** |
| Do you need multi-protocol (NFS + SMB) or ONTAP features? | **FSx for NetApp ONTAP** |
| Do you need extreme performance with ZFS? | **FSx for OpenZFS** |

---

## 7. Real-World Scenarios

### NollywoodFlix (Lagos) — Video Processing

**Requirement:** 12 parallel EC2 Linux instances transcode movie uploads. All instances must read from the same raw folder and write to shared volume.

| Step | Action |
|------|--------|
| 1 | Upload raw videos to S3 |
| 2 | Files copy to **Multi-AZ EFS** |
| 3 | 12 workers process EFS data in parallel |
| 4 | Output pushed to CloudFront |

**Solution:** Amazon EFS — NFS for Linux, multi-AZ resilience, concurrent access.

### Pinnacle Finance (Abuja) — Legacy Windows Migration

**Requirement:** 350 staff using Windows file shares. Need Active Directory integration, SMB compatibility, self-service restores.

**Solution:** FSx for Windows File Server
- Multi-AZ high availability with native SMB
- Integrates with AWS Managed Microsoft AD
- DFS Namespaces keep existing folder paths (`\\pinnacle\finance`)
- VSS Shadow Copies for self-service restores

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "You can mount EFS on Windows EC2 instances" | ❌ EFS is **officially supported on Linux only**. Windows has optional NFS client but not recommended. For Windows workloads → FSx for Windows. |
| "FSx for Lustre Scratch is fine for long-term data analysis" | ❌ **Scratch has no replication.** Data lost if host fails. Long-term jobs require **Persistent** deployment. |
| "EFS Bursting mode provides maximum speed for small file systems" | ❌ Bursting throughput scales with size (50 KB/s per GB). Small file systems = low throughput. Use **Provisioned** or **Elastic** for small, fast datasets. |
| "EFS can serve as root boot volume for EC2" | ❌ EC2 requires **block storage (EBS/Instance Store)** for root volume. EFS mounts as **additional network volume** after boot. |
| "EFS Max I/O has the lowest latency" | ❌ **General Purpose** has lowest latency (optimized for latency-sensitive apps). Max I/O adds metadata latency for scalability. |

---

## 9. Knowledge Check — Practice Questions

### Q1: Video Transcoding Across 3 AZs

**Scenario:** 20 video encoding EC2 Linux instances across 3 AZs. All need simultaneous read/write access to same video library.

**Correct answer:** **Amazon EFS with General Purpose performance mode**

| Option | Why wrong/right |
|--------|-----------------|
| EBS Multi-Attach (io2) | Restricted to single AZ |
| **Amazon EFS** | ✅ Managed NFS for parallel Linux across AZs |
| One large EBS gp3 shared across instances | EBS is 1-to-1, not shared |
| FSx for Windows | SMB protocol not native to Linux transcoding |

### Q2: High-Performance Genomics Processing

**Scenario:** 72-hour genome sequencing job on 100 EC2 Linux instances. Need sub-millisecond storage with 500,000+ IOPS, backed by S3.

**Correct answer:** **FSx for Lustre (Persistent) linked to S3 bucket**

| Option | Why wrong/right |
|--------|-----------------|
| Amazon EFS Max I/O | Not designed for 500K+ IOPS sub-millisecond |
| EBS io2 attached to each instance | No shared file system across 100 instances |
| **FSx for Lustre** | ✅ Purpose-built for HPC: parallel access, sub-millisecond, direct S3 integration |
| S3 accessed directly via SDK | Object storage, not file system; higher latency |

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| EFS protocol | NFSv4 (Linux only) |
| FSx for Windows protocol | SMB (Windows + Linux via SMB) |
| FSx for Lustre best for | HPC, ML training, genomics, rendering |
| EFS One Zone savings | 47% cheaper than Regional |
| EFS default performance mode | General Purpose (lowest latency) |
| EFS throughput that scales automatically | Elastic (recommended) |
| FSx Lustre for long-term HA jobs | Persistent (not Scratch) |
| FSx Lustre direct S3 integration | Yes — bi-directional sync |
| Windows Active Directory integration | FSx for Windows File Server |
| Cannot serve as root boot volume | EFS (needs block storage) |

---

## 11. One Sentence to Remember

> **EFS = Linux-only NFS shared across AZs (General Purpose for low latency, Max I/O for scale, One Zone for 47% savings). FSx for Windows = SMB + Active Directory (Windows file shares). FSx for Lustre = HPC parallel file system (Persistent for long-term, Scratch for temporary, direct S3 integration). EFS cannot be root boot volume. Choose based on OS, protocol, and performance needs.**
