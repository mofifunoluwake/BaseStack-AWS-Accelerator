# ☁️ Week 7 · Day 1: Serverless & AWS Lambda

**Module:** Serverless  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — The Shift From Servers to Functions

### Traditional Cloud (EC2)

| Challenge | Impact |
|-----------|--------|
| Rent virtual servers | Pay for provisioned capacity 24/7 |
| Install and configure software | Manual setup for every server |
| Apply OS patches | Security updates on your time |
| Keep instances running | Idle compute = wasted money |
| Manual scaling | Requires intervention or complex config |

**The problem:** You pay whether your app has 1 user or 1 million. Idle compute is wasted money.

### Serverless with Lambda

| Benefit | Why |
|---------|-----|
| **Zero server management** | AWS handles infrastructure |
| **Pay only when code executes** | Idle = $0 |
| **Automatic scaling built in** | Zero to thousands without configuration |
| **First 1M requests/month free** | Free Tier covers most learning projects |

**The shift:** Write code → choose runtime → AWS handles everything else. Pay only for milliseconds your code actually runs.

---

## 2. Key Concepts — The Vocabulary of Serverless

| Term | Definition |
|------|------------|
| **Lambda Function** | Your code packaged with a chosen runtime (Python, Node.js, Java, Go, Ruby, .NET, or custom container). AWS runs it only when triggered. Like a vending machine — idle until someone presses a button. |
| **Event Source / Trigger** | The thing that starts your Lambda function running. Common triggers: S3 upload, API Gateway request, SQS message, CloudWatch scheduled event, DynamoDB Stream, Cognito event. |
| **Execution Role (IAM Role)** | IAM Role Lambda assumes while running. Defines which AWS services and resources your function can access (S3, DynamoDB, SNS, etc.). Without correct role → AccessDenied errors. |

---

## 3. Lambda Limits, Concurrency & Cold Starts (Heavily Tested)

### Timeout — 15 Minutes Hard Limit

| Limit | Detail |
|-------|--------|
| Maximum execution time | **15 minutes** — hard limit, cannot be increased |
| After timeout | AWS forcefully terminates execution |
| For longer jobs (video transcoding, ML inference, multi-hour pipelines) | Use **ECS, EC2, or AWS Batch** instead |

**Exam trap:** The exam loves to test when NOT to use Lambda. Long-running job = not Lambda.

### Concurrent Executions

| Concept | Detail |
|---------|--------|
| Definition | How many copies of your function can run in parallel at the same time |
| Default limit | 1,000 per Region (soft limit — can be increased via support request) |
| If limit hit during spike | Additional invocations are **throttled** (rejected) |
| **Reserved Concurrency** | Guarantee capacity for a specific function |
| **Provisioned Concurrency** | Pre-warm containers to eliminate cold starts |

### Cold Start vs Warm Start

| | Cold Start | Warm Start |
|--|------------|------------|
| **When** | First invocation after being idle | Subsequent invocations |
| **What happens** | AWS provisions new microVM, loads code, initializes runtime | Reuses already-warm container |
| **Latency** | +100–500ms extra | Near-instant |
| **Solution** | Provisioned Concurrency (pre-warms containers) | N/A |

**Exam tip:** Cold starts affect **latency**, not correctness. Provisioned Concurrency solves cold starts but costs money even when idle — trade-off between performance and cost.

---

## 4. How Lambda Executes — The Full Lifecycle

| Stage | What happens |
|-------|--------------|
| 1 | **Event triggers** — S3 upload, API call, etc. |
| 2 | **Container boots** — (cold start) AWS provisions microVM |
| 3 | **Handler runs** — your code executes (what you're billed for) |
| 4 | **Response returned** — result sent back to trigger |
| 5 | **Container stays warm** — free, makes subsequent calls fast |

---

## 5. Billing Model — What You Actually Pay

| Component | Cost |
|-----------|------|
| **Requests** | $0.20 per 1 million requests |
| **Compute duration** | Memory allocated × execution time (rounded to nearest ms) |
| **Free Tier** | First 1M requests/month + 400,000 GB-seconds/month free |

**Example cost:** 128MB function running for 100ms → ~$0.0000021 per invocation.

**Why this pricing matters:**
- Traditional servers charge for provisioned capacity (pay whether busy or idle)
- Lambda charges for actual work done
- At very high sustained traffic (100M+ requests/month), EC2 Reserved Instances may be cheaper

**Exam tip:** Lambda is ideal for spiky, unpredictable, or low-volume workloads. For steady, high-volume workloads, EC2 Reserved Instances may be more cost-effective.

---

## 6. Configuration Limits & Common Triggers

### Configuration Limits

| Setting | Limit |
|---------|-------|
| Maximum timeout | **15 minutes** (hard limit) |
| Memory allocation | 128 MB – 10 GB |
| Ephemeral storage (`/tmp`) | 512 MB – 10 GB |
| Deployment package (zip) | 50 MB (250 MB unzipped) |
| Default concurrency | 1,000 per Region (soft limit) |
| Environment variables | 4 KB total |

### Common Event Sources (Triggers)

| Trigger | Use case |
|---------|----------|
| **API Gateway** | Build serverless backends and microservices exposed to internet |
| **S3** | File uploads — image processing, document parsing, media transcoding |
| **SQS / SNS** | Decoupled, asynchronous processing at scale |
| **CloudWatch Events** | Scheduled cron jobs — data cleanup, report generation, health checks |
| **DynamoDB Streams** | React to row-level changes in real-time |
| **Cognito / ALB** | Authentication triggers, serverless web applications |

**Supported runtimes:** Python, Node.js, Java, Go, Ruby, .NET, custom runtimes via container image (up to 10 GB).

---

## 7. Serverless Architecture — How the Pieces Connect
Browser/Mobile App
       │
       ▼ (HTTPS)
  ┌─────────────┐
  │ API Gateway │ ← front door, auth, validation
  └─────────────┘
       │
       ▼ (triggers)
  ┌─────────────┐
  │   Lambda    │ ← business logic
  │   Function  │
  └─────────────┘
       │
       ├──────────┬──────────┐
       ▼          ▼          ▼
  ┌─────────┐ ┌─────┐ ┌──────────────┐
  │DynamoDB │ │ S3  │ │ CloudWatch   │
  │(read/   │ │(store│ │Logs (logging)│
  │ write)  │ │files)│ │              │
  └─────────┘ └─────┘ └──────────────┘
  
  | Layer | Service | Purpose |
|-------|---------|---------|
| 1 | Browser/Mobile App | Client sends HTTPS request |
| 2 | API Gateway | Front door, authentication, validation |
| 3 | Lambda Function | Business logic execution |
| 4a | DynamoDB | Read/write data |
| 4b | S3 | Store files |
| 4c | CloudWatch Logs | Logging and monitoring |


**Why this architecture wins:**
- Fully serverless — no EC2 instances, no containers, no OS to patch
- Scales automatically — zero to thousands of requests per second
- Pay only when users make requests
- Loose coupling — each component can be updated independently

This is the most common serverless pattern on the SAA-C03 exam.

---

## 8. Real-World Nigerian Scenario — OluPay Fintech, Lagos

**Problem:** Every time a user completes a payment, they receive a PDF receipt stored in S3. Team needs to parse PDF, extract transaction details, store for analytics, and send confirmation — without managing servers.

### Without Lambda (EC2)

| Problem | Impact |
|---------|--------|
| EC2 running 24/7 polling S3 | ₦ wasted on idle compute |
| Manual scaling during traffic spikes | Overwhelmed at peak hours (9am/5pm) |
| Security patches, OS updates, monitoring | Team spends time on infrastructure, not features |

### With Lambda (Serverless)

| Step | Action |
|------|--------|
| 1 | User pays → receipt PDF uploaded to S3 |
| 2 | S3 event automatically triggers Lambda |
| 3 | Lambda parses PDF, extracts transaction data |
| 4 | Writes to DynamoDB → sends SNS confirmation (SMS) |
| 5 | Function idles → **zero cost until next payment** |

### Results

| Metric | Value |
|--------|-------|
| Idle cost | **₦0** — no idle servers burning money |
| Transactions per day | 50,000 — scales automatically with zero configuration |
| Steps | 5 steps from payment to confirmation — fully serverless |

---

## 9. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| **"Lambda can run for 1 hour"** | ❌ Maximum execution time = **15 minutes** (hard limit). Longer jobs → ECS, EC2, or Batch. |
| **"Lambda is always cheaper than EC2"** | ❌ At very high sustained volume (100M+ requests/month), **EC2 Reserved Instances** may be cheaper. Lambda = ideal for spiky/unpredictable/low-volume. |
| **"/tmp directory is shared between invocations"** | ❌ `/tmp` is **local to ONE execution environment**. Each concurrent invocation gets isolated space. Need shared data? Use S3, DynamoDB, or ElastiCache. |
| **"Lambda has no execution time limit"** | ❌ Hard limit = **15 minutes** per invocation. Cannot be increased. |

**Exam strategy for Lambda questions:**
1. Is the runtime under 15 minutes?
2. Is the traffic pattern spiky or sustained?
3. Does it need shared state between invocations?

If any answer is "no" — Lambda may not be the right choice.

---

## 10. Practice Question (SAA-C03 Style)

**Scenario:** A Lagos fintech needs to process user KYC documents uploaded to S3. Documents arrive in bursts around 9am and 5pm. The team wants zero server management and minimal cost.

**Which is the BEST solution?**

| Option | Answer |
|--------|--------|
| EC2 Auto Scaling + CloudWatch alarm | ❌ Servers to manage. Idle cost between bursts. |
| **S3 Event → AWS Lambda** | ✅ **Correct** — S3 triggers Lambda on upload. Scales from zero during quiet periods (no idle cost). Handles burst traffic automatically. Zero servers. |
| AWS Fargate always-on container | ❌ Always-on = idle cost between bursts. More complex. |
| RDS trigger + EC2 worker fleet | ❌ Overly complex. Database triggers not designed for this. Servers to manage. |

**Answer:** B — S3 Event → AWS Lambda matches both requirements perfectly.

---

## 11. Quick Reference Card

| Question | Answer |
|----------|--------|
| Maximum Lambda timeout | **15 minutes** (hard limit) |
| Memory range | 128 MB – 10 GB |
| Ephemeral storage (`/tmp`) | 512 MB – 10 GB |
| Default concurrency per Region | 1,000 (soft limit) |
| Free Tier requests | 1 million/month |
| Cold start latency | +100–500ms |
| Cold start solution | Provisioned Concurrency (costs $ even when idle) |
| Longer than 15 minutes? | Use ECS, EC2, or Batch (not Lambda) |
| Share data between invocations? | S3, DynamoDB, ElastiCache (not `/tmp`) |
| When is EC2 cheaper than Lambda? | Very high sustained volume (100M+ requests/month) |
| Most common trigger for serverless APIs | API Gateway |

---

## 12. One Sentence to Remember

> **Lambda = serverless, event-driven, pay-per-execution (first 1M requests/month free). Hard limits: 15 minutes timeout, 10GB memory, 1,000 concurrency default. Cold start adds 100–500ms latency — use Provisioned Concurrency to eliminate (costs $ when idle). For longer jobs → ECS/EC2/Batch. For shared state between invocations → S3/DynamoDB (not `/tmp`). Lambda for spiky/unpredictable workloads; EC2 Reserved may be cheaper for very high sustained volume.**