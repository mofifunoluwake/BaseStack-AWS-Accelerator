# ☁️ Week 2 · Day 3: SCPs & AWS Organizations

**Module:** IAM & Security  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

As companies grow on AWS, managing security across dozens of accounts using individual IAM policies becomes impossible. **AWS Organizations** solves this by consolidating multiple AWS accounts under a single management umbrella — with centralized control, unified billing, and hierarchical policy enforcement.

**Service Control Policies (SCPs)** are the enforcement mechanism. They act as a **permissions ceiling** on every account in your organization. Even an IAM Administrator in a member account cannot exceed what an SCP allows.

**Without SCPs:** A rogue developer could disable audit logging, spin up resources in prohibited regions, or incur uncapped costs.

**With SCPs:** You prevent all of this at scale. Centralized. Enforced. Unbypassable.

**For Nigerian fintechs and banks:** SCPs are how security teams enforce data residency (e.g., no resources outside `af-south-1` or `eu-west-1`), prevent cost overruns, and satisfy CBN and ISO 27001 audit requirements.

---

## 2. The Core Pieces (Know These Definitions)

### AWS Organizations

| What it is | A free global AWS service that groups multiple AWS accounts under one management account |
|------------|----------------------------------------------------------------------------------------|
| What it enables | Consolidated billing, centralized governance, hierarchical policy enforcement |
| Is it free? | Yes — no additional charge for Organizations |

### Management Account

| What it is | The AWS account that creates the organization |
|------------|-----------------------------------------------|
| What it can do | Invite/create member accounts, create OUs, attach SCPs, view consolidated bills |
| **Critical rule** | The Management Account is **NEVER restricted by SCPs** — even if an SCP is attached to the root |

### Organisational Unit (OU)

| What it is | A logical folder inside the organization hierarchy |
|------------|----------------------------------------------------|
| How you use it | Group accounts by function — Production OU, Development OU, Security OU |
| How inheritance works | SCPs attached to an OU are automatically inherited by **all accounts and nested OUs** beneath it |

### Service Control Policy (SCP)

| What it is | A JSON policy document that defines the **MAXIMUM** permissions available in an account or OU |
|-------------|----------------------------------------------------------------------------------------------|
| Does it grant access? | **NO.** SCPs never grant access — they only restrict |
| Effective permission formula | `IAM Policy (Allow)` AND `SCP (Allow)` → both must allow for access to be granted |
| What happens if SCP denies | The call fails — even if the IAM policy is `AdministratorAccess` |

### FullAWSAccess SCP

| What it is | The default SCP automatically attached to every root, OU, and account when you enable SCPs |
|------------|-------------------------------------------------------------------------------------------|
| What it does | Allows all actions on all services |
| Why it exists | SCPs start **permissive** — you add restrictions on top, rather than whitelisting from scratch |

---

## 3. How It Works — Step by Step

### Step 1: Set Up AWS Organizations

From your **Management Account**, navigate to AWS Organizations and click Create Organization. AWS creates a **Root** container automatically. Then:

1. Invite or create member accounts
2. Create **Organisational Units (OUs)** to group accounts logically (Production OU, Development OU, Security OU)
3. Understand that accounts inherit **all** SCPs attached to their parent OU

### Step 2: Write & Attach an SCP

SCPs use the **exact same JSON syntax** as IAM policies.

1. Write your policy document (e.g., deny all EC2 actions outside `af-south-1` and `eu-west-1`)
2. Navigate to AWS Organizations → Policies → Service Control Policies
3. Create the policy
4. Attach it to the target OU or account
5. All accounts under that OU **immediately inherit** the restriction — no per-account action needed

### Step 3: How Permission Evaluation Works

Every AWS API call in a member account goes through **two evaluations**:

| Evaluation | Who controls it |
|------------|-----------------|
| Is the action allowed by the **SCP**? | Organization admin (centralized) |
| Is the action allowed by the **IAM policy**? | Account admin (per-account) |

**Both must say YES for the request to succeed.**

If the SCP says Deny → call fails — even if the IAM policy is AdministratorAccess.

**Think of it as a two-lock door:** You need both keys (SCP key + IAM key) to open it.

---

## 4. Two SCP Strategies (Exam Critical)

### Deny List (Recommended Default)

| How it works | Keep the default `FullAWSAccess` SCP attached. Add explicit Deny statements for what's forbidden. |
|--------------|---------------------------------------------------------------------------------------------------|
| What's allowed | Everything not explicitly denied |
| Best for | Most organizations starting with SCPs — easier to manage |
| Overhead | Low — only forbidden actions need documentation |

### Allow List (High-Security / Regulated)

| How it works | Remove `FullAWSAccess`. Replace with explicit Allow for only approved services. |
|--------------|--------------------------------------------------------------------------------|
| What's allowed | Only services explicitly allowed — everything else is implicitly denied |
| Best for | Financial services (CBN compliance), healthcare, defense, highly regulated industries |
| Overhead | High — every new AWS service must be explicitly approved before use |

**Exam insight:** If a question describes a bank or healthcare company, lean toward Allow List. If it describes a startup or general enterprise, Deny List is more common.

---

## 5. SCP JSON Example — Deny Non-Approved Regions

This SCP prevents resource creation outside approved regions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonApprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotIn": {
          "aws:RequestedRegion": [
            "eu-west-1",
            "af-south-1"
          ]
        }
      }
    }
  ]
}
