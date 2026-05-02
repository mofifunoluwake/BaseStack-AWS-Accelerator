# ☁️ Week 2 · Day 4: CloudTrail & Audit Logging

**Module:** IAM & Security  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture (Why This Matters)

Every action in your AWS environment — every API call, console login, resource creation, or deletion — leaves a digital fingerprint. **AWS CloudTrail** captures those fingerprints and stores them as tamper-evident log records.

**Without CloudTrail, you cannot answer critical questions:**
- Who deleted that S3 bucket?
- Which IAM user changed the security group rule?
- What IP address was used to log in before the data breach?

CloudTrail turns your AWS account from a black box into a fully auditable system.

**Real-world context:** In 2022, a major cloud data breach took weeks longer to investigate because the organization hadn't enabled CloudTrail in all regions. The evidence existed — but wasn't captured. That mistake cost millions in fines.

**For Nigerian financial institutions:** The Central Bank of Nigeria (CBN) Cybersecurity Framework requires financial institutions to maintain audit trails of all system activity. ISO 27001 also explicitly requires access and activity logging. CloudTrail is AWS's implementation of this requirement.

---

## 2. The Core Pieces (Know These Definitions)

### AWS CloudTrail

| What it is | A regional AWS service that continuously records all API calls and account activity across your AWS infrastructure |
|------------|----------------------------------------------------------------------------------------------------------------|
| What sources it captures | Console, CLI, SDK, other AWS services |
| Output format | JSON log files stored in an S3 bucket |
| Is it free? | Management events are free for one copy per region. Data events and Insights cost extra. |

### Trail

| What it is | A configuration in CloudTrail that defines WHERE logs go, WHICH regions to capture, and WHICH event types to include |
|------------|---------------------------------------------------------------------------------------------------------------------|
| What happens without a Trail | Only the last 90 days of management events are retained (Event History) — no persistence, no alerts |
| How to get long-term retention | Create a Trail that delivers logs to an S3 bucket |

### CloudTrail Event

| What it is | A single JSON record of one API call or AWS action |
|------------|-----------------------------------------------------|
| What each event contains | WHO (principal ARN), WHAT (event name), WHEN (timestamp), WHERE (source IP + region), and outcome (success/failure) |

### Event History

| What it is | A free, always-on view of the last 90 days of management events in the CloudTrail console |
|------------|-------------------------------------------------------------------------------------------|
| Does it require configuration? | No — it's always there |
| **Critical warning** | It is NOT a substitute for a proper Trail. It doesn't persist beyond 90 days and cannot trigger alerts. |

### S3 Bucket (Log Destination)

| What it is | Where CloudTrail delivers log files (you specify the bucket) |
|------------|--------------------------------------------------------------|
| Requirements | Bucket policy must grant CloudTrail permission to write |
| Delivery speed | ~15 minutes after the event |
| File format | Compressed `.gz` JSON |

### CloudTrail Lake

| What it is | A managed event data lake that lets you run SQL queries against CloudTrail events |
|------------|-----------------------------------------------------------------------------------|
| What it replaces | Extracting logs from S3 and loading into another database |
| Retention | Up to 7 years |
| Best for | Forensic investigations across multiple accounts and regions |

---

## 3. CloudTrail Event Types — What Gets Recorded

CloudTrail captures three distinct categories. Understanding them is essential for both the exam and cost management.

### Management Events (Control Plane)

| What they are | Operations that configure or manage AWS resources |
|---------------|----------------------------------------------------|
| Enabled by default? | **Yes** — in all Trails |
| Examples | `CreateBucket`, `DeleteBucket`, `RunInstances`, `TerminateInstances`, `AttachRolePolicy`, `ConsoleLogin` |
| Exam insight | This is what most governance and compliance solutions rely on |

### Data Events (Data Plane) — High Volume

| What they are | Object-level operations on resources |
|---------------|--------------------------------------|
| Enabled by default? | **NO** — must be explicitly turned on |
| Why they're off by default | S3 data events can generate millions of log entries per day (cost) |
| Examples | `GetObject`, `PutObject`, `DeleteObject` (S3); `InvokeFunction` (Lambda); `GetItem`, `PutItem` (DynamoDB) |
| Exam trap | Questions often test whether you know data events are OFF by default. Required for forensics on S3 data access. |

### CloudTrail Insights (Anomaly Detection)

| What it is | Uses machine learning to detect unusual API activity patterns |
|-------------|---------------------------------------------------------------|
| Is it free? | No — additional cost |
| What it detects | Unusual spike in write API volume, unusual burst of error rates, deviations from baseline |
| Prerequisite | Requires standard Management Events as a baseline |
| Exam insight | Insights ≠ standard event logging. It's an anomaly detection layer on top. |

---

## 4. How It Works — Step by Step

### Step 1: Create a Trail & Configure Delivery

1. In the CloudTrail console, create a new Trail
2. Give it a name
3. Choose the S3 bucket where logs will be delivered
4. **Critical:** Select "Apply trail to all regions" to capture events across every AWS region
5. Optionally enable CloudWatch Logs delivery for real-time monitoring
6. Optionally enable log file integrity validation — one checkbox that provides forensic-grade tamper detection

### Step 2: Choose Event Types to Capture

- **Management Events** — enabled by default (free)
- **Data Events** — decide if you need them (costs money, high volume)
  - For a production fintech: enable S3 Data Events to capture every object access
  - Enable Lambda Data Events if you need visibility into function invocations
- **CloudTrail Insights** — enable if you need anomaly detection

**Be deliberate:** Data Events can generate very high log volumes and cost. Don't enable them unless you have a specific need.

### Step 3: Events Flow Through the Pipeline

1. Any principal makes an API call
2. CloudTrail captures it within ~15 minutes
3. A compressed JSON log file is written to your S3 bucket
4. Folder structure: `BucketName/AWSLogs/AccountID/CloudTrail/Region/YYYY/MM/DD/`
5. If CloudWatch Logs is configured, the event streams there in near real-time
6. CloudWatch Metric Filters evaluate the stream and trigger Alarms when specific patterns match

---

## 5. Real CloudTrail Event (JSON Breakdown)

Here's what an actual event looks like:

```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "arn": "arn:aws:iam::123456789:user/developer1",
    "userName": "developer1"
  },
  "eventTime": "2024-09-15T14:32:01Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "DeleteBucket",
  "awsRegion": "eu-west-1",
  "sourceIPAddress": "197.210.52.8",
  "requestParameters": {
    "bucketName": "fintech-customer-data"
  },
  "responseElements": null
}
