# ☁️ Week 5 · Day 3: S3 Security & Access Control

**Module:** Storage  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

S3 is where companies store payment receipts, customer uploads, application logs, compliance reports, database backups, and sensitive documents. One weak permission can expose years of accumulated data in seconds.

**The beginner mistake:** "The app cannot load the file, so I will make the bucket public." This solves one error and creates a data breach. Public buckets are indexed by search engines, scanned by bots, and appear in breach disclosure reports.

**The engineer mindset:** Start from the user, role, app, action, object path, and time limit. Grant the smallest permission that works — and no more.

**Ask yourself:** Which is worse for a business — losing old files or exposing private files? S3 security controls exist to protect against both outcomes simultaneously.

---

## 2. The S3 Access Decision Flow

When someone requests an object, S3 evaluates several layers before returning Allow or Deny.

| Step | What S3 evaluates |
|------|-------------------|
| 1 | **Who is asking?** — IAM user, IAM role, AWS service, or anonymous internet user |
| 2 | **Public Access Guardrail** — Block Public Access settings evaluated first. If enabled, public access rejected before bucket policy is consulted |
| 3 | **Policy Evaluation** — IAM policy + bucket policy evaluated together. **Explicit Deny always wins** over any Allow |
| 4 | **Object Ownership** — Modern buckets use "Bucket owner enforced" (ACLs disabled). Access lives in IAM + bucket policies only |
| 5 | **Final Result** — Object returned or AccessDenied |

**Core Rule:** Keep the bucket private. Give access through identities, policies, and temporary links — never through public bucket settings.

---

## 3. Bucket Policies — The Language of S3 Permissions

A bucket policy is a JSON resource policy attached directly to an S3 bucket. It answers four questions.

| Element | What it defines | Example |
|---------|-----------------|---------|
| **Principal** | Who gets access | AWS account ARN, IAM role ARN, `*` (avoid unless necessary) |
| **Action** | What can they do | `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket` |
| **Resource** | Where does this apply | Bucket ARN (`arn:aws:s3:::my-bucket`) or bucket/* ARN (`arn:aws:s3:::my-bucket/*`) |
| **Condition** | When does it apply | Enforce TLS, restrict by IP, require encryption headers |

### Common Security Patterns

| Pattern | Policy element |
|---------|----------------|
| Deny uploads without encryption | `s3:x-amz-server-side-encryption` condition |
| Deny non-HTTPS requests | `aws:SecureTransport: false` |
| Allow app role to read specific prefix | Scope `Resource` to `bucket/prefix/*` |
| Explicit Deny for public access | `Principal: "*"` with `Effect: "Deny"` |

**Student question:** A school portal should let students read only their own result PDFs. Which part of the policy separates one student from another?

**Answer:** Use a condition with `s3:prefix` or embed the student ID in the resource ARN path (e.g., `bucket/results/{student-id}/*`).

---

## 4. Block Public Access & Object Ownership

Modern S3 design starts with public access blocked and ACLs disabled. These two settings form your first line of defense.

### Block Public Access (BPA) — Four Settings

| Setting | What it blocks |
|---------|----------------|
| Block public ACLs on new objects | Prevents new ACLs from granting public access |
| Block public ACLs on existing objects | Ignores existing public ACLs |
| Block public bucket policies | Prevents bucket policies from granting public access |
| Restrict public cross-account access | Blocks cross-account public access |

**When BPA is enabled, public access stays blocked even if someone writes a public policy.** This is your emergency guardrail.

### Object Ownership — Bucket Owner Enforced (Modern Default)

| What it does | Effect |
|--------------|--------|
| Disables ACLs entirely | ACLs are ignored |
| Bucket owner is sole access controller | No cross-account ACL grants |
| All access logic in IAM + bucket policies | Simplified security model |

**Public Website Exception:** Only disable BPA when the bucket is explicitly meant to serve public static content. When you do, add a strict `GetObject` policy scoped to the minimum required.

---

## 5. Versioning, Delete Markers & MFA Delete

Versioning protects you from accidental overwrite and simple deletion. MFA Delete adds an additional layer of protection.

### How Versioning Works
PUT (v1) → PUT (v2) → DELETE → Delete Marker (object hidden, versions remain)
↓
Remove Delete Marker → v2 accessible again


| Operation | What happens |
|-----------|--------------|
| PUT object | New numbered version created |
| DELETE object | Adds a **Delete Marker** — object hidden, data NOT gone |
| Remove Delete Marker | Object accessible again instantly |

### MFA Delete

When enabled on a versioned bucket, MFA Delete requires the **root user's MFA device** to:

- Permanently delete a specific object version
- Change the versioning state of the bucket

This prevents a compromised IAM user from silently destroying all historical versions of critical data.

### Important Warnings

| Warning | Why |
|---------|-----|
| **Cost** | Old versions consume storage indefinitely → use lifecycle rules to expire noncurrent versions |
| **Versioning cannot be fully disabled** | Can be suspended, but existing versions remain accessible |
| **Delete = Delete Marker** | Data is not gone — just hidden |

**Exam trap:** A delete operation on a versioned bucket creates a delete marker — the data is not gone, just hidden.

---

## 6. Encryption Choices — Simple, Audited, or Customer-Managed

S3 encrypts all new objects by default using SSE-S3. Different compliance needs call for different strategies.

| Encryption Type | How it works | Audit trail | Best for |
|----------------|--------------|-------------|----------|
| **SSE-S3** | AWS manages keys (AES-256). Zero config. | No | Normal private buckets — encryption without audit needs |
| **SSE-KMS** | AWS KMS customer master keys. CloudTrail audit logs. | **Yes** — shows who used which key, when | Financial records, healthcare data, compliance reports |
| **SSE-C** | You supply the key with every PUT. AWS doesn't store it. | No | Regulatory requirements demanding AWS never touches your keys |
| **Client-Side** | You encrypt before upload. AWS never sees plaintext. | No | Maximum security, maximum complexity |

### Quick Decision Guide

| Need | Choose |
|------|--------|
| Encryption only (no audit) | SSE-S3 (default, free) |
| Audit trails + key rotation control | SSE-KMS |
| AWS cannot store your encryption keys | SSE-C |
| You want total control before upload | Client-Side |

**Exam clue:** SSE-KMS = "encryption with audit trail" or "customer-managed keys."

---

## 7. Presigned URLs — Share Private Objects Safely

A presigned URL grants temporary, limited access to **one** private S3 object without making the bucket public.

### How It Works

| Step | What happens |
|------|--------------|
| 1 | **Private bucket** — Object stays private. Block Public Access remains ON. |
| 2 | **App signs URL** — IAM principal with `s3:GetObject` creates time-limited link using AWS SDK |
| 3 | **Client downloads** — Client uses URL directly. No AWS credentials needed. |
| 4 | **URL expires** — S3 returns AccessDenied automatically. No cleanup needed. |

| Attribute | Detail |
|-----------|--------|
| Expiry range | 1 second to **7 days** (max) |
| After expiry | 403 Access Denied automatically |
| Good use case | Send client invoice PDF for 15 minutes. Keep every other object private. |
| Bad shortcut | Turning off BPA because one user needs one file. Use presigned URLs instead. |

**Exam trap:** Presigned URL ≠ public access. It's temporary, scoped, and identity-based.

---

## 8. Scenario Workshop — KudiFi Payment Receipts

**Context:** Lagos fintech stores payment receipts, chargeback evidence, and daily settlement reports in S3. Must keep data private, allow users to download their own receipts, support recovery of deleted reports, and meet audit requirements.

### Requirements & Solutions

| Requirement | Solution |
|-------------|----------|
| Receipts must stay private | **Block Public Access ON** — all four settings enabled |
| Users download only their own receipts | **Prefix-scoped policy** — `receipts/${cognito:sub}/*` |
| Finance team reads all settlement reports | **IAM role with policy** scoped to `reports/*` |
| Deleted reports must be recoverable | **Versioning ON** + lifecycle rule to expire noncurrent versions after 90 days |
| Audit-friendly encryption with key trails | **SSE-KMS** for settlement reports (CloudTrail audit). Receipts use SSE-S3 for simplicity. |
| Users receive files securely | **Presigned URLs** — 15-minute expiry, users never see bucket structure |

**Strong Answer Summary:**
- Block Public Access ON
- ACLs disabled
- Versioning ON
- SSE-KMS for reports, SSE-S3 for receipts
- Presigned URLs for user downloads

---

## 9. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Public bucket needs only a bucket policy" | ❌ Need **BOTH**: disable Block Public Access AND bucket policy granting `*` |
| "Explicit Deny can be overridden by a more specific Allow" | ❌ **Explicit Deny always wins** — overrides any Allow, regardless of specificity |
| "Deleting an object in a versioned bucket removes it permanently" | ❌ Adds a **Delete Marker** — data remains until version is permanently deleted |
| "SSE-S3 provides audit trails for key usage" | ❌ Only **SSE-KMS** provides CloudTrail key usage logs |
| "Presigned URLs make the bucket public" | ❌ **No** — temporary, scoped, identity-based. Bucket stays private. |
| "MFA Delete can be required for IAM users" | ❌ **Root user's MFA device only** — protects critical version operations |

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| First line of defense against public exposure | Block Public Access (evaluated before bucket policy) |
| Modern access control model (no ACLs) | Bucket Owner Enforced |
| Delete on versioned bucket creates | Delete Marker (data not gone) |
| How to restore versioned object | Remove the Delete Marker |
| Encryption with CloudTrail audit | SSE-KMS |
| Default S3 encryption | SSE-S3 (AES-256, free) |
| Temporary access to private object | Presigned URL (max 7 days) |
| Prevents IAM user from deleting versions | MFA Delete (requires root MFA) |
| Explicit Deny vs Allow | Explicit Deny always wins |

---

## 11. Lab Practice Checklist

| Task | Done |
|------|------|
| Create private bucket — Block Public Access ON from start | ☐ |
| Upload files under `receipts/` and `temp/` prefixes | ☐ |
| Enable versioning — upload same file twice, confirm two versions | ☐ |
| Delete a file — observe delete marker, then remove to restore | ☐ |
| Generate presigned URL with 60-second expiry — test it stops working | ☐ |
| Share design on LinkedIn/X | ☐ |

---

## 12. One Sentence to Remember

> **Block Public Access is your emergency guardrail (evaluated first). Bucket policies control access with Principal + Action + Resource + Condition. Versioning protects from deletion (Delete Marker, not permanent). MFA Delete requires root MFA to delete versions. SSE-KMS provides audit trails; SSE-S3 is default free encryption. Presigned URLs grant temporary access without making buckets public. Explicit Deny always wins.**
