# ☁️ Week 2 · Day 2: IAM Policies — Writing & Reading Permissions

**Module:** IAM & Security  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

IAM Policies are how permissions are granted in AWS. Every single action you take — launching an EC2 instance, uploading to S3, invoking a Lambda function — is checked against IAM policies.

**The default-deny model:** If no policy explicitly allows an action, it is denied. AWS is secure out of the box. Access must be intentionally granted. Nothing is accidentally open.

You don't need to write complex JSON from scratch. AWS provides hundreds of pre-built managed policies. But you absolutely need to **read** policies, understand what they permit, and know when to use managed vs custom policies.

---

## 2. The Core Pieces (Policy Types)

### Managed Policies

| What they are | Pre-built policies created and maintained by AWS (or by you as customer managed) |
|---------------|---------------------------------------------------------------------------------|
| **AWS Managed** | Created by AWS — e.g., `AmazonS3ReadOnlyAccess`, `AdministratorAccess` |
| **Customer Managed** | Created by you — you maintain them, but they are reusable across multiple identities |
| **Best for** | Common use cases, standard roles, things you need to attach to multiple users/groups |

### Inline Policies

| What they are | Policies embedded directly inside a **single** user, group, or role |
|---------------|---------------------------------------------------------------------|
| **Reusable?** | No — they live only where you create them |
| **Best for** | One-off permissions that should never be reused (rare — usually avoid) |
| **Risk** | Hard to audit because permissions are scattered instead of centralized |

**Rule of thumb:** Use managed policies (AWS or customer) 95% of the time. Inline policies are for very specific edge cases.

---

## 3. The Policy Structure (The JSON Blueprint)

Every IAM policy has the same skeleton. Four core elements:

| Element | What it does | Example |
|---------|--------------|---------|
| **Effect** | Allow or Deny | `"Effect": "Allow"` |
| **Action** | Which AWS API calls | `"Action": "s3:GetObject"` or `"Action": ["s3:GetObject", "s3:ListBucket"]` |
| **Resource** | Which specific resources | `"Resource": "arn:aws:s3:::my-bucket/*"` |
| **Condition** | Optional — adds restrictions | `"Condition": {"IpAddress": {"aws:SourceIp": "203.0.113.0/24"}}` |

### The Policy Structure in JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::company-app-bucket",
        "arn:aws:s3:::company-app-bucket/*"
      ]
    }
  ]
}
