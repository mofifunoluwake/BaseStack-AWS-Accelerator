# ☁️ Day 3: The Shared Responsibility Model

**Module:** Cloud Foundations  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

The #1 cause of cloud breaches is **not AWS messing up**. It's **customers messing up their own configurations**.

Remember the Capital One breach (2019, 100 million records exposed)? AWS had perfectly secured the physical data centres. But Capital One left its digital front door wide open — misconfigured firewall, bad access controls. AWS built a fortress. The customer left the gate unlocked.

**The Shared Responsibility Model is a legal contract that says:**  
- AWS secures **the physical stuff** (buildings, servers, cables)  
- You secure **your digital stuff** (data, passwords, who can access what)

No confusion. No finger-pointing. Clear lines.

For Nigerian organisations worried about "losing control" in the cloud: you don't lose control. You just shift *what* you control. You still hold all the digital keys.

---

## 2. The Core Concept (Just Two Buckets)

| Bucket | Who owns it? | What's inside? |
|--------|--------------|----------------|
| **Security OF the Cloud** | AWS | Physical data centres, server hardware, network cables, virtualisation layer, global fibre backbone |
| **Security IN the Cloud** | YOU (the customer) | Your data, your application code, your OS patches (on EC2), your firewall rules, your IAM users, your encryption settings |

**One sentence summary:**  
> *AWS locks the building. You lock your apartment door inside that building.*

---

## 3. How the Line Shifts (Most Important for the Exam)

Not every AWS service is the same. The more AWS *manages* for you, the more responsibility AWS takes.

### Example 1: EC2 (IaaS — You manage almost everything)

| Responsibility | Who does it? |
|----------------|--------------|
| Physical data center | AWS |
| Server hardware | AWS |
| Virtualization layer | AWS |
| **Operating system patches** | **YOU** |
| **Application code** | **YOU** |
| **Firewall rules (Security Groups)** | **YOU** |
| **Data encryption** | **YOU** |

### Example 2: RDS (Managed database — AWS manages more)

| Responsibility | Who does it? |
|----------------|--------------|
| Physical data center | AWS |
| Server hardware | AWS |
| Virtualization layer | AWS |
| **Operating system patches** | **AWS** (you don't touch this) |
| **Database engine patches** | **AWS** (automatic) |
| **Who can log into the database** | **YOU** |
| **What data is stored** | **YOU** |
| **Encryption settings** | **YOU** |
| **Network access rules** | **YOU** |

### Example 3: Lambda (Serverless — AWS manages almost everything)

| Responsibility | Who does it? |
|----------------|--------------|
| Everything about servers | AWS |
| Runtime environment | AWS |
| **Your function code** | **YOU** |
| **Your function's dependencies/libraries** | **YOU** |

**The pattern:**  
The further you move from "I rent a server" to "I just upload code," the less you manage. But you *always* manage your data, your users, and your access controls.

---

## 4. The Three Types of Controls (Exam Loves This)

| Control Type | Meaning | Example |
|--------------|---------|---------|
| **AWS-only** | Only AWS can do this | Physical security, hardware replacement |
| **Customer-only** | Only you can do this | Your IAM users, your data classification |
| **Shared** | Both parties do this | Patch management, configuration management, security awareness training |

**Exam trap:** When you see a question about patch management or configuration management, the answer is almost always **"Shared Control"** — both AWS and the customer have pieces of this.

---

## 5. Real-World Nigerian Scenario

**A healthcare startup in Lagos stores patient medical records on Amazon S3.**

- **AWS's job:** Keep the hard drives in a locked, CCTV-monitored, access-controlled facility with fire suppression. If a disk physically fails, AWS replaces it. You never think about hardware.

- **The startup's job:**  
  - Encrypt the data  
  - Create IAM policies so only authorised doctors can read records  
  - Turn on S3 access logging (so they can see who touched what)  
  - **Never accidentally make the bucket public** (this is the #1 mistake)

If that startup leaves the bucket public and patient records leak? That's 100% on them. AWS will not automatically close that door.

---

## 6. What "Inherited Controls" Means

**Fancy term:** Inherited Controls  
**What it actually means:** You don't need to audit AWS's data centre yourself. AWS already pays independent auditors to do that. You just download the reports.

**How you prove compliance to CBN (Central Bank of Nigeria) or any regulator:**  
1. Go to **AWS Artifact** (a self-service portal)  
2. Download the SOC reports, ISO certifications, and PCI compliance reports  
3. Hand them to your auditor  

You never fly to Cape Town. You never walk through an AWS data centre. You just download the proof.

---

## 7. Exam Traps (These WILL show up)

| Wrong belief | Actual truth |
|--------------|---------------|
| "AWS will fix my misconfigured S3 bucket for me" | ❌ No. AWS gives you the tool. You set the permissions. If you make it public, that's your breach. |
| "RDS means AWS handles everything" | ❌ No. AWS handles OS and DB patching. You handle users, data, encryption, and network rules. |
| "Patch management is 100% AWS's job" | ❌ No. It's a **Shared Control**. On EC2, you patch the OS. On RDS, AWS patches OS/DB, but you patch your app. On Lambda, you patch your dependencies. |
| "I can inspect an AWS data centre myself" | ❌ Absolutely not. You use AWS Artifact to download audit reports. |
| "If hardware fails, I replace it" | ❌ Never. Hardware is **Security OF the Cloud** = AWS's job. |

---

## 8. The One Thing to Memorise for the Exam

> **AWS is ALWAYS responsible for the physical infrastructure layer — hardware, facilities, global networking.  
> Customers are ALWAYS responsible for their data classification and IAM access controls.**

And when you see **patch management** or **configuration management** in a question — those are **Shared Controls**.

---

## 9. One Sentence to Remember

> **AWS secures the building. You secure your apartment inside it. The more managed the service, the larger AWS's apartment becomes — but you always lock your own door.**
