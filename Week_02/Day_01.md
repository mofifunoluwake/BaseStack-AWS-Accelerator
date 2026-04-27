# ☁️ Week 2 · Day 1: IAM Deep Dive — Roles vs Users

**Module:** IAM & Security  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

IAM is the foundation of AWS security. It controls **who** can do **what** in your AWS account. It's also one of the most heavily tested areas on the SAA-C03 exam — Domain 1 (Design Secure Architectures) is 30% of the entire exam, and IAM sits right in the middle of it.

**One critical thing to know:** IAM is a **global** service. Users, groups, roles, and policies you create are available across all Regions. No need to recreate anything when you switch Regions.

---

## 2. The Core Pieces (Identity Types)

### Root Account

| What it is | The email address you used to create the AWS account |
|------------|-----------------------------------------------------|
| How powerful | Unlimited power — can close the account, change billing, bypass every restriction |
| What to do with it | Enable MFA immediately, then lock it away. Never use for daily work. |
| Can you restrict root? | NO. Nothing can restrict root — not even policies. This is why you MFA it and forget it. |

### IAM User

| What it is | An identity for a **person** or **application** |
|------------|------------------------------------------------|
| Credentials | Permanent — username/password for console, access keys for CLI/API |
| Default permissions | NONE. You must attach policies to give them access |
| Best practice | One person = one IAM User. Never share credentials. |

### IAM Group

| What it is | A collection of IAM users |
|------------|--------------------------|
| Why use it | Attach policies to the **group**, not individual users. All members inherit permissions automatically. |
| Example | Create a `Developers` group with EC2 + Lambda permissions. Add any developer to the group — they get access instantly. Remove them from the group — access disappears. |

### IAM Role

| What it is | An identity for **AWS services** (EC2, Lambda, etc.) |
|------------|------------------------------------------------------|
| Credentials | **None.** Roles don't have passwords or long-term access keys. They get **temporary** credentials from STS. |
| How it works | A service "assumes" the role and gets temporary keys that expire automatically |
| Best practice | Never put access keys on EC2. Attach a role instead. |

---

## 3. IAM Users vs IAM Roles — The Critical Distinction

This is the most exam-tested difference in IAM. Memorize this table.

| | IAM User | IAM Role |
|--|----------|----------|
| **Credentials** | Permanent (password + access keys) | Temporary (STS issues keys that expire) |
| **Who uses it** | Humans (developers, admins, analysts) | AWS Services (EC2, Lambda, etc.) |
| **When to use** | Someone needs to log into console or run CLI daily | A service needs permissions to access other services |
| **Has password?** | Yes | No |
| **Can be rotated?** | Yes (manually) | Yes (automatically — every time it's assumed) |

**The one sentence to remember:**
> *Users are for people. Roles are for services.*

---

## 4. How IAM Roles Work Under the Hood

Every IAM Role has **two policies** attached to it:

### Trust Policy (WHO can assume this role)

Defines which service or account is allowed to call `AssumeRole`.

**Example:** A trust policy for an EC2 role says `ec2.amazonaws.com` is allowed to assume this role. That means EC2 instances can "become" this role.

### Permissions Policy (WHAT can this role do once assumed)

Defines the actual actions allowed — like reading from S3, writing to CloudWatch, etc.

**How it works step by step:**

1. You create an IAM Role with:
   - **Trust Policy:** `ec2.amazonaws.com` can assume this
   - **Permissions Policy:** Read from `my-bucket` and write to CloudWatch

2. You attach the role to an EC2 instance

3. EC2 automatically calls the **STS (Security Token Service)** `AssumeRole` API

4. STS issues **temporary credentials** (access key ID + secret access key + session token)

5. The application on EC2 retrieves these credentials automatically from the instance metadata service at `169.254.169.254`

6. The credentials expire after a set duration (default 1 hour, max 12 hours)

**The key insight:** No developer ever touches, sees, or handles these credentials. They are delivered automatically. This is why roles are far more secure than long-term access keys stored in code.

---

## 5. IAM Policies — The JSON That Defines Permissions

A policy is a JSON document with three required parts:

| Component | What it means | Example |
|-----------|---------------|---------|
| **Effect** | Allow or Deny | `"Effect": "Allow"` |
| **Action** | What operation | `"Action": "s3:GetObject"` |
| **Resource** | Which specific resource | `"Resource": "arn:aws:s3:::my-bucket/*"` |

**Example policy allowing read access to a specific S3 bucket:**

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::client-data-bucket/*"
}
