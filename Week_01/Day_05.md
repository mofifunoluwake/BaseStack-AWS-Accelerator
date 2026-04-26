# ☁️ Day 5: IAM & Account Security Setup

**Module:** Cloud Foundations  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

The AWS Management Console is your cockpit. It's the web interface where you'll create, manage, and monitor everything — EC2 servers, S3 buckets, databases, all of it. But before you touch *any* service, you need to do two things:

1. **Secure your account** (IAM, MFA, root account rules)
2. **Set up billing alerts** (so you never get a surprise $500 bill)

Cloud bills can surprise even experienced engineers. The moment you create a resource, the meter starts running. Billing alerts are like checking your phone balance before making a call — basic professionalism.

**For a bootcamp student:** David forgot to terminate his EC2 instance after a lab exercise. No billing alert. Got a $47 surprise bill. A $5 alert would have emailed him within 2–3 days, plenty of time to shut it down. This habit saves hundreds of dollars.

---

## 2. The Console Basics (Your AWS Cockpit)

| Component | What it does | Pro tip |
|-----------|--------------|---------|
| **AWS Management Console** | Web browser interface at `console.aws.amazon.com` — command centre for all 200+ AWS services | Bookmark it |
| **Region Selector** | Dropdown in top-right corner showing which Region you're working in | **Always check this before creating resources** — services are Region-specific! |
| **Global Search Bar** | Search for any service by name | Faster than scrolling through menus |
| **Account Menu** | Top-right — shows your account ID, billing, logout | Where you'll find "My Billing Dashboard" |

**Critical rule:** IAM and Billing are **global** — they work across all Regions. But everything else (EC2, S3, RDS) lives in a specific Region. If you create an EC2 instance in `us-east-1` and then switch to `eu-west-2`, you won't see it.

---

## 3. Billing Alarms: Your Financial Safety Net

There are **two ways** to get alerted about spending. You should set up both.

### Option 1: CloudWatch Billing Alarm (Simple)

| Step | What to do |
|------|-------------|
| 1 | Switch to **us-east-1** (N. Virginia) — billing data lives ONLY in this Region |
| 2 | Go to CloudWatch → Alarms → Billing → Create Alarm |
| 3 | Set metric: **Estimated Charges** |
| 4 | Set threshold: **Greater than $5** |
| 5 | Create an SNS topic (Simple Notification Service) |
| 6 | Add your email address |
| 7 | Check your email and confirm the subscription link |

**Why $5?** It's low enough that you'll know within days of leaving something running, but high enough that it won't trigger on tiny free-tier usage.

### Option 2: AWS Budgets (More Powerful)

| Feature | What it does |
|---------|--------------|
| **Budget types** | Actual spend, forecasted spend, or Reserved Instance utilisation |
| **Multiple thresholds** | Alert at 50%, 80%, and 100% of your budget |
| **Automatic** | Can trigger alerts to SNS (email, SMS, or Lambda) |

**The free tier:** AWS Budgets sends up to 5 alerts per month for free. CloudWatch Billing Alarms charges after the free tier. For learning, either works.

### One More Alarm: Free Tier Usage Alert

There's a **separate** built-in alert under AWS Billing settings that warns when you're approaching Free Tier limits. Enable this too.

**Exam trap:** Billing and cost data are ONLY in `us-east-1`. If you create a billing alarm in any other Region, you will see no data. Always switch to us-east-1 first.

---

## 4. IAM Deep Dive: Who Can Do What?

IAM (Identity and Access Management) is how you control who can access your AWS account and what they can do.

### The Root Account (God Mode — Handle With Care)

| What it is | Why it's dangerous |
|------------|-------------------|
| The email address you used to create the AWS account | Can close the account, delete all data, change any settings |
| Unlimited power — no restrictions | If compromised, you lose everything |

**Non-negotiable rules:**
1. Enable **MFA** on root account immediately
2. **Lock away** the root password (password manager, printed and in a safe)
3. **Never** use root for daily tasks
4. Create an IAM user with admin permissions and use that instead

### IAM Identities (The "Who")

| Identity | What it is | Example |
|----------|------------|---------|
| **IAM User** | An individual person or application | `david.adeleke`, `ci-server` |
| **IAM Group** | A collection of users with the same permissions | `Developers`, `Managers`, `ReadOnly` |
| **IAM Role** | Temporary permissions assumed by AWS services | An EC2 instance needs a role to access S3 — **never put access keys on EC2** |

### IAM Policies (The "What")

Policies are JSON documents that define permissions. They have three parts:

| Component | What it means | Example |
|-----------|---------------|---------|
| **Effect** | Allow or Deny | `"Effect": "Allow"` |
| **Action** | What operation | `"Action": "s3:GetObject"` |
| **Resource** | Which specific resource | `"Resource": "arn:aws:s3:::my-bucket/*"` |

**Example Policy (Allow user to read objects in a specific S3 bucket):**

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-company-bucket/*"
}
