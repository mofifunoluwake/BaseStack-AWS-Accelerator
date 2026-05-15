# ☁️ Week 4 · Day 5: EC2 Storage — EBS, Instance Store & Snapshots

**Module:** Compute  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Your EC2 instance is the computer. Storage decides what happens to your data when the instance stops, fails, or gets replaced.

| Storage type | What it is | Persistence |
|--------------|-----------|-------------|
| **EBS** | External SSD over the network | ✅ Survives stop/start |
| **Instance Store** | Physical SSD attached to host | ❌ Lost on stop/terminate |
| **Snapshots** | Point-in-time backup in S3 | ✅ Permanent until deleted |

---

## 2. The Nigerian Analogy

| Storage type | Analogy |
|--------------|---------|
| **EBS** | External hard drive — your real files are saved. Even if laptop shuts down, files remain. |
| **Instance Store** | Temporary whiteboard — fast, but data disappears if system stops. |
| **Snapshots** | Backup copy of your external drive kept safely in an archive. |

---

## 3. EBS Essentials (Elastic Block Store)

| Attribute | Detail |
|-----------|--------|
| **What it is** | Persistent block storage. Survives instance stop and restart. |
| **Use for** | OS disks, databases, app files, uploads — anything you cannot afford to lose |
| **AZ-bound** | Lives in one Availability Zone. Snapshot → restore to move across AZs. |
| **Encryption** | Uses AWS KMS. Enable when creating the volume. |
| **Root EBS** | May delete on termination by default — change this setting deliberately. |

**The critical question:** If the instance stops at midnight, which storage still has your customer data in the morning? **EBS.**

---

## 4. EBS Volume Types — Match Workload to Volume

| Type | Best for | Avoid for |
|------|----------|-----------|
| **gp3** | General apps, boot volumes, web apps, "cost-effective SSD" | Highest sustained IOPS needed |
| **io2** | Critical DB, guaranteed IOPS, SAP/Oracle, Multi-Attach | Ordinary or cost-sensitive workloads |
| **st1** | Logs, big data, streaming reads, high throughput HDD | Boot volumes, random I/O |
| **sc1** | Cold data, rarely accessed, lowest-cost block storage | Performance or frequent access |

**Quick rule:**
- `gp3` → most workloads (default)
- `io2` → critical high IOPS (databases)
- `st1` → sequential throughput (logs)
- `sc1` → cold data (archives)

---

## 5. Instance Store — Fast but Dangerous

Physically attached to the host — very fast, but tied to the host's lifecycle.

### Good Use Cases
- Cache and temp files
- Scratch processing and buffers
- Distributed systems that replicate data elsewhere

### Bad Use Cases
- Customer uploads or payment records
- Database files or source code
- Any single copy of important data

### Data Survival by Event

| Event | Instance Store data |
|-------|---------------------|
| Reboot | ✅ Persists |
| Stop | ❌ Lost |
| Hibernate | ❌ Lost |
| Terminate | ❌ Lost |

**The question to ask:** "Can we afford to lose this when the instance stops?" If no → Instance Store is wrong.

---

## 6. Snapshots — Backup & Recovery

| Attribute | Detail |
|-----------|--------|
| **What it is** | Point-in-time backup of EBS volume |
| **Where stored** | S3 (behind the scenes — you don't see them directly) |
| **Incremental** | First capture = full backup. Later captures = only changed blocks. |
| **Use for** | Backup, recovery, migration across AZs or Regions |

### Snapshot Lifecycle
Create volume (type, size, AZ, encryption)
↓
Attach & mount (format, mount, update /etc/fstab)
↓
Use the volume
↓
Snapshot (automate with AWS Backup or DLM)
↓
Restore & recover (to same or different AZ/Region)


**Critical rule:** A backup you have never restored is only a hope, not a recovery plan. **Test restores regularly.**

---

## 7. Encryption — How to Encrypt EBS

| Rule | Detail |
|------|--------|
| **When to enable** | When creating the volume |
| **Cannot encrypt in place** | If volume is unencrypted, you cannot encrypt it directly |
| **The process** | Snapshot → copy snapshot with encryption enabled → restore new encrypted volume |

**Exam trap:** The question will describe an unencrypted volume that needs encryption. The correct answer is **not** "enable encryption on the volume" — it's **"snapshot → copy encrypted → restore."**

---

## 8. DeleteOnTermination — The Root Volume Trap

| Setting | What happens when instance terminates |
|---------|---------------------------------------|
| **Enabled (default)** | Root EBS volume is DELETED |
| **Disabled** | Root EBS volume is preserved |

**Best practice:** Set `DeleteOnTermination = OFF` for data volumes. Automate snapshots instead. For root volumes, be deliberate — do you want to keep or delete?

---

## 9. Reference Architecture — Safe EC2 Storage Design

| Design rule | Why |
|-------------|-----|
| Databases on EBS (never Instance Store) | Data must survive stop/start |
| Use io2 for guaranteed IOPS | Critical databases need predictable performance |
| Enable KMS encryption at volume creation | Compliance and security |
| Automate snapshots with AWS Backup | Recovery and migration |
| Cache on Instance Store only | Disposable data only |
| Set DeleteOnTermination deliberately | Prevent accidental data loss |

---

## 10. Real-World Scenario (PayQuick Lagos)

**Context:** Fintech processes card payments on EC2 — transaction logs, receipt uploads, and fraud-check temp data. Dev instances stop at 2 AM to save cost.

| Component | Storage choice | Why |
|-----------|---------------|-----|
| Transaction database | Encrypted EBS (gp3 or io2) | Must survive stop. Customer data cannot be lost. |
| Receipt uploads | Encrypted EBS | Never Instance Store — payments data. |
| Fraud-check temp data | Instance Store only | Only if loss has zero customer impact. Cache only. |
| DeleteOnTermination | OFF for data volumes | Automate snapshots instead. |

**Trap avoided:** Putting payment data on Instance Store because "it's fast" — it vanishes on stop. Critical failure.

---

## 11. EBS vs Instance Store — Side by Side

| Feature | EBS | Instance Store |
|---------|-----|----------------|
| **Persistence** | ✅ Survives stop/start | ❌ Lost on stop/terminate |
| **Speed** | Good | Very fast (physically attached) |
| **Use for** | OS, databases, app files, customer data | Cache, temp files, buffers |
| **AZ scope** | Tied to one AZ (move via snapshot) | Tied to host AZ |
| **Encryption** | Yes (KMS) | No |
| **Snapshots** | Yes | No |
| **Billing** | GB-month + IOPS | Included in instance price (but lost on stop) |

---

## 12. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Instance Store data persists through stop" | ❌ **Lost on stop, hibernate, terminate.** Only reboot preserves it. |
| "You can encrypt an unencrypted EBS volume directly" | ❌ **Cannot encrypt in place.** Snapshot → copy encrypted → restore. |
| "EBS volumes can be moved directly to another AZ" | ❌ **EBS is AZ-bound.** Must snapshot → restore in target AZ. |
| "All EBS snapshots are full backups" | ❌ **Incremental after the first.** Only changed blocks are stored. |
| "Terminating an instance always keeps the root EBS" | ❌ **DeleteOnTermination defaults to ON.** Root EBS is deleted unless you disable it. |
| "Instance Store is fine for customer data" | ❌ **Never.** Data disappears on stop. Use encrypted EBS. |

---

## 13. Practice Questions (Quick Check)

| Question | Answer |
|----------|--------|
| Database must survive instance stop — EBS or Instance Store? | **EBS** — persists through stop |
| How to move EBS from us-east-1a to us-east-1b? | Snapshot → restore in target AZ |
| Normal web app boot volume — which type? | **gp3** — default, cost-effective SSD |
| Which event preserves Instance Store data? | **Reboot only** (stop/hibernate/terminate lose it) |
| Can you directly encrypt an unencrypted EBS volume? | **No.** Snapshot → copy encrypted → restore |

---

## 14. Quick Reference Card

| Question | Answer |
|----------|--------|
| Survives instance stop? | EBS = Yes, Instance Store = No |
| Default root EBS behavior on termination? | Deleted (DeleteOnTermination = ON) |
| How to move EBS to another AZ? | Snapshot → restore in target AZ |
| EBS encryption method? | AWS KMS |
| How to encrypt an unencrypted volume? | Snapshot → copy encrypted → restore |
| Snapshots are full or incremental? | Incremental after first |
| Best EBS type for most workloads? | gp3 |
| Best EBS type for critical high IOPS DB? | io2 |
| Best for cold, rarely accessed data? | sc1 |
| Instance Store survives reboot? | Yes |
| Instance Store survives stop? | No |

---

## 15. One Sentence to Remember

> **EBS = persistent (survives stop). Instance Store = fast but lost on stop/terminate (cache only). Snapshots = incremental backups stored in S3. EBS is AZ-bound — move via snapshot. Cannot encrypt unencrypted volume in place — snapshot → copy encrypted → restore. Set DeleteOnTermination deliberately (default ON = root EBS deleted).**
