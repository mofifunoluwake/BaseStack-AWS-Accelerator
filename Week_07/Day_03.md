# ☁️ Week 7 · Day 3: SQS & SNS Messaging

**Module:** Serverless  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why Decoupling Matters

**The Problem: Tight Coupling**

Without decoupling: if your checkout service calls your payment service directly and payment goes down — checkout goes down too. A flash sale spike sends 50,000 requests/second and your backend crashes. You cannot scale one part without affecting everything else.

**The Analogy:** A busy Danfo bus stop in Lagos. Passengers (messages) arrive and wait. The driver (processing service) picks them up when ready — nobody crashes into each other. That's decoupling.

**The AWS Solution:** SQS and SNS act as a buffer and message router between components. Each service works independently — if one slows down or fails, the others keep running.

| Without SQS/SNS | With SQS/SNS |
|-----------------|---------------|
| Direct calls create cascading failures | Messages queue safely, processed when ready |
| Everything is entangled | Scale each service independently |
| Hard to scale | Decoupled by design |

---

## 2. SQS — Amazon Simple Queue Service

A fully managed message queue that stores messages and delivers them to **one consumer at a time** (point-to-point delivery). Think of it as a digital inbox that holds messages until a worker is ready.

### Queue Types

| Feature | Standard Queue | FIFO Queue |
|---------|----------------|------------|
| **Ordering** | Best-effort only | Strict order guaranteed |
| **Delivery** | At-least-once | Exactly-once |
| **Throughput** | Nearly unlimited | 300 msg/sec (3,000 with batching) |
| **Use for** | High-volume pipelines, order not critical | Financial transactions, order processing |

### Key Configuration

| Setting | Value | Purpose |
|---------|-------|---------|
| **Max message size** | 256 KB | Single message limit |
| **Retention** | Up to 14 days | Messages persist if not processed |
| **Visibility timeout** | 30 seconds (default) | Message hidden while processing (prevents double processing) |
| **Max batch receive** | 10 messages | Pull multiple messages at once |

---

## 3. SNS — Amazon Simple Notification Service

A fully managed pub/sub messaging service that sends **one message to many subscribers simultaneously** (broadcast delivery). One publisher, unlimited subscribers — delivered in under 1 second.

### How SNS Works

| Step | Action |
|------|--------|
| 1 | Payment app publishes event to SNS Topic (e.g., "PaymentEvents") |
| 2 | SNS broadcasts to all subscribers simultaneously |
| 3 | Subscribers receive in parallel |

### Who Can Subscribe to SNS?

| Subscriber Type | Use Case |
|-----------------|----------|
| **SQS Queue** | Fan out to processing pipelines |
| **Lambda** | Trigger serverless functions instantly |
| **Email / SMS** | Alert humans directly |
| **HTTP/HTTPS** | Call external webhooks |
| **Mobile Push** | iOS and Android notifications |

### Key Scale Numbers

| Metric | Value |
|--------|-------|
| Max subscribers per topic | 12.5 million |
| Delivery speed | Under 1 second |
| Message storage | **None** — delivered immediately or lost |

**Critical difference:** SNS does NOT store messages. If no subscriber is available at publish time, the message is lost.

---

## 4. SQS vs SNS — When to Use Which

| Feature | SQS | SNS |
|---------|-----|-----|
| **Delivery** | One consumer per message | Many subscribers simultaneously |
| **Pattern** | Point-to-point (Direct Message) | Publish/Subscribe (Broadcast) |
| **Storage** | Messages stored up to 14 days | NOT stored — delivered immediately |
| **Use when** | Reliable async task processing with retry | Notify multiple systems at once |
| **Examples** | Order queues, email jobs, image resize pipelines | Payment alerts, monitoring, app notifications |

**Quick Rule:**
- Tasks needing retry/durability → **SQS**
- Alerts to multiple services → **SNS**
- Both together → **SNS Fan-Out Pattern**

---

## 5. SQS Deep Dive — Key Features

### Dead-Letter Queue (DLQ)

| Feature | Detail |
|---------|--------|
| **What it does** | If a message fails after `maxReceiveCount` retries, it moves to DLQ |
| **Purpose** | Catch broken ("poison pill") messages without losing them |
| **Essential for** | Production systems |

**Exam Tip:** "What happens to messages that fail repeatedly?" → They go to the Dead-Letter Queue.

### Long Polling

| Feature | Short Polling | Long Polling |
|---------|---------------|--------------|
| **How it works** | Checks every second (wasteful) | Waits up to 20 seconds for message |
| **Cost** | Higher (many empty responses) | Cuts API calls by up to 90% |

### Message Deduplication (FIFO only)

| Feature | Detail |
|---------|--------|
| **How it works** | Uses Deduplication ID to detect duplicates |
| **Window** | Same ID within 5 minutes → second copy discarded |
| **Use case** | Prevents double payments or duplicate emails |

### Delay Queues

| Feature | Detail |
|---------|--------|
| **What it does** | Delays message delivery by 0–900 seconds (15 minutes) |
| **Use case** | Wait for downstream service to be ready (e.g., database record saved first) |

---

## 6. SNS Deep Dive — Key Features

### Message Filtering

| Feature | Detail |
|---------|--------|
| **What it does** | Each subscriber has a filter policy — only receives relevant messages |
| **Example** | Fraud-detection Lambda only receives messages where `amount > 500,000 NGN` |

### Message Encryption

| Feature | Detail |
|---------|--------|
| **What it does** | Server-side encryption using AWS KMS |
| **Requirement** | Fintech and healthcare must encrypt sensitive data at rest |

### FIFO Topics

| Feature | Detail |
|---------|--------|
| **What it does** | Guarantees message order and deduplication |
| **Limitation** | Only supported with **SQS FIFO** subscribers (not email or SMS) |

### The Fan-Out Pattern (AWS Recommended)

| Step | Action |
|------|--------|
| 1 | Payment App publishes **once** to SNS Topic |
| 2 | SNS fans out to **multiple SQS queues** |
| 3 | Each SQS queue serves one downstream service |

**Flow:** `Payment App → SNS Topic → Ledger Queue | Fraud Queue | SMS Queue`

**Benefits:**
- Publish once, fan out to many consumers
- Each consumer processes at its own pace
- DLQs on each queue for resilience
- Recommended in AWS Well-Architected Framework

---

## 7. Real-World Nigerian Scenario — OluPay Fintech

**Context:** Nigerian fintech processing ₦2.5 billion in transfers daily. Old architecture had payment API calling ledger, fraud engine, SMS service, and audit logger directly. During salary rush, SMS service slowed down — entire payment system stalled waiting for it.

### Step 1 — Decouple with SNS

| Before | After |
|--------|-------|
| Payment API called each service directly (blocking) | Payment API publishes event to SNS topic |
| Waited for slowest service to respond | Responds to user in under 200ms |
| Cascading failures | No waiting |

### Step 2 — Fan Out to SQS Queues

| Queue | Purpose |
|-------|---------|
| **LedgerQueue** | Accounting service updates balances |
| **FraudQueue** | Lambda checks for anomalies |
| **SMSQueue** | Sends confirmations via Termii |

### Step 3 — Resilience with DLQ

| Feature | Benefit |
|---------|---------|
| DLQ on each queue | If SMS service goes down during spike, messages queue safely |
| Recovery | When service recovers, it processes backlog — no transactions lost |

**Discussion:** Which part of OluPay's new architecture would you have built first — and why?

---

## 8. Exam Traps (Memorize These)

| Misconception | Truth |
|---------------|-------|
| "SNS stores messages for retry" | ❌ **SNS does NOT store messages.** If no subscriber available at publish time, message is lost. Use SQS for durability and retry. |
| "Standard queue guarantees message order" | ❌ Only **FIFO queues** guarantee ordering. Standard queues = best-effort only. Question mentions "order" or "sequence" → FIFO. |
| "One Lambda = one SQS message at a time" | ❌ Lambda can batch-process up to **10,000 messages** from SQS in parallel. |
| "FIFO queues have the same throughput as Standard" | ❌ Standard = nearly unlimited. FIFO = 300 TPS (3,000 with batching). For high-volume pipelines → Standard + deduplication logic in code. |
| "SNS message is guaranteed to be received" | ❌ SNS has **no storage**. Message is lost if subscriber unavailable. SQS + DLQ for guaranteed processing. |

---

## 9. Quick Reference Card

| Question | Answer |
|----------|--------|
| SQS delivery pattern | 1:1 — one consumer per message |
| SNS delivery pattern | 1:many — broadcast to all subscribers |
| SQS max retention | 14 days |
| SNS message storage | **None** — delivered immediately or lost |
| SQS FIFO throughput | 300 msg/sec (3,000 with batching) |
| Long polling wait time | Up to 20 seconds |
| Dead-Letter Queue purpose | Capture messages that fail repeatedly |
| SNS + SQS pattern | Fan-Out (publish once, many queues) |
| "Message order required" → which queue? | FIFO Queue |
| "High throughput, order not critical" → which queue? | Standard Queue |
| "Notify multiple systems at once" → which service? | SNS |
| "Reliable task processing with retry" → which service? | SQS |

---

## 10. Practice Task

**Draw the SQS + SNS fan-out architecture for OluPay from memory — label every component:**

- SNS Topic (PaymentEvents)
- SQS Queue 1 (LedgerQueue) → Accounting Service
- SQS Queue 2 (FraudQueue) → Fraud Detection Lambda
- SQS Queue 3 (SMSQueue) → SMS Service (Termii)
- DLQ attached to each queue

---

## 11. One Sentence to Remember

> **SQS = point-to-point (one consumer, stores messages up to 14 days, DLQ for failed messages). SNS = pub/sub (one message, many subscribers, NO storage — message lost if subscriber unavailable). FIFO queues guarantee order but lower throughput. Long polling cuts costs by 90%. SNS + SQS Fan-Out = publish once, many queues — AWS Well-Architected pattern.**