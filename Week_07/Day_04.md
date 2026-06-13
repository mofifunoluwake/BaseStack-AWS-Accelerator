# ☁️ Week 7 · Day 4: Step Functions & Workflow Orchestration

**Module:** Serverless  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — Why Workflow Orchestration Matters

Modern cloud applications are rarely a single Lambda function. A real-world payment flow might involve: validate → fraud check → debit → notify → audit.

**The Problem: Tight Coupling**

Lambda-to-Lambda chains fail silently. Without orchestration, there's no built-in way to retry individual steps, track which step failed, or measure how long each step took. You end up writing hundreds of lines of manual retry and timeout logic inside each function.

**The Solution: Step Functions**

A fully managed serverless orchestration service that coordinates AWS services into visual, auditable workflows using state machines defined in JSON (Amazon States Language).

**Analogy:** Think of Step Functions like a flowchart that AWS can actually run. Each box is a step — AWS handles the arrows between them, including retries, error handling, and logging.

**What You Gain:**

| Benefit | Why It Matters |
|---------|----------------|
| Full visual workflow diagram | See the entire process at a glance |
| Built-in retry logic | Configurable intervals, no code needed |
| Execution history logged to CloudWatch | Up to 90 days of audit trail |
| Loose coupling | Each step independent and testable |
| No glue code | AWS manages state transitions |

**Exam Note:** Orchestration patterns appear in approximately **30% of SAA-C03 scenario questions**.

---

## 2. Key Concepts — Core Vocabulary

| Term | Definition |
|------|------------|
| **State Machine** | The complete workflow blueprint — a graph of states and transitions written in Amazon States Language (ASL, JSON format) |
| **State** | A single step in the workflow — task, decision, wait, parallel branch, or end point |
| **Execution** | One live run of the state machine for a specific input payload. Each execution gets a unique ARN and full event log. |
| **Task State** | Does real work — calls Lambda, writes to DynamoDB, sends SNS, or integrates with any AWS service |
| **Choice State** | Decision branch — if/else routing based on payload data |
| **Wait State** | Pauses workflow for a set duration or until a specific timestamp |
| **Parallel State** | Runs multiple branches simultaneously; waits for all to finish |
| **Retry & Catch** | Built-in error handling: auto-retry failed states; Catch routes to fallback handlers |

---

## 3. The Seven State Types (Exam Critical)

| State Type | What It Does | SAA-C03 Signal / When to Use |
|------------|--------------|------------------------------|
| **Task** | Calls a service — Lambda, DynamoDB, SNS, SQS, ECS | "invoke Lambda" / "call service" — any real processing step |
| **Choice** | Evaluates conditions and routes to different branches | "based on the result" / "different paths" — if/else routing |
| **Wait** | Pauses execution for duration or timestamp | "wait 24 hours" / "human approval window" — scheduled delays |
| **Parallel** | Runs multiple branches simultaneously; waits for all to finish | "simultaneously" / "at the same time" — independent overlapping tasks |
| **Map** | Iterates over an array, running same steps for each item | "for each item" / "process the list" — batch processing |
| **Succeed / Fail** | Terminal states — Succeed ends successfully; Fail triggers Catch handlers | "workflow complete" / "raise an error" — explicit end points |

**Exam Insight:** The SAA-C03 loves to test the difference between **Parallel** (run branches at same time) and **Map** (iterate over an array). Parallel is for independent concurrent tasks; Map is for processing a list of items one-by-one or in batch.

---

## 4. How Step Functions Executes: The Payment Flow

**Example:** A customer initiates a payment — here's how the state machine processes it:

```text
Start → Validate KYC → Fraud Check → Debit & Notify → Emit Receipt
           ↓ (if fails)        ↓ (if score > 80)
          Reject              Fail State → SNS Alert

```      
Each state receives the JSON output from the previous state, processes it, and passes the enriched payload forward.

## 5. Standard vs. Express Workflows — Know the Difference

| Feature | Standard Workflow | Express Workflow |
|---------|-------------------|------------------|
| Max Duration | Up to 1 year | Maximum 5 minutes |
| Execution Model | Exactly-once — each state executes precisely once | At-least-once — states may retry; Lambda must be idempotent |
| Execution Rate | 2,000 executions/second | 100,000 executions/second |
| Pricing Model | Per state transition | Per execution + duration |
| Execution History | Full audit log retained for 90 days | CloudWatch Logs only (no console history) |
| Best Use Cases | Payments, approvals, long-running, audit-required | IoT events, streaming data, high-volume short-lived |

### When to Choose Standard

- Financial transactions requiring exactly-once semantics
- Multi-step approval workflows that may take days
- Any process requiring a full 90-day audit trail
- Workflows with human approval wait states

### When to Choose Express

- High-volume event processing (IoT sensor streams)
- Short-lived workflows under 5 minutes
- Scenarios where at-least-once execution is acceptable
- Cost-sensitive, high-throughput pipelines

**Critical Exam Point:** If a question mentions "millions of IoT events" or "high-throughput streaming," the answer is almost always Express Workflows. Standard's 2,000/sec limit is a hard ceiling.

---

## 6. Architecture — How the Pieces Connect

| Layer | Component | Role |
|-------|-----------|------|
| Trigger | API Gateway, EventBridge, or StartExecution API | Initiates workflow with JSON payload |
| Orchestration | State Machine (ASL JSON) | Blueprint defining states, transitions, retries, catches |
| Compute | Task States → Lambda / Services | Business logic execution (Step Functions orchestrates, does NOT compute) |
| Routing | Choice State → Branch Routing | Evaluates conditions, routes to different paths |
| Logging | Succeed / Fail → CloudWatch Logs | Every execution event logged for audit |

**Key Architectural Principle:** Step Functions **orchestrates** — it does **not** compute. All business logic lives inside Lambda functions or other AWS services. Step Functions only manages order, state, retries, and routing.

---

## 7. Real-World Nigerian Scenario — OluPay Loan Disbursement

**Context:** Lagos fintech offering instant nano-loans to market traders. Trader applies for ₦50,000 loan. System must: verify identity → check credit score → assess risk → calculate rate → transfer funds → notify customer — all in seconds, with full CBN-compliant audit trail.

### Workflow Steps

| Step | State Type | Action | Error Handling |
|------|------------|--------|----------------|
| 1 | Task | Lambda calls NIMC API to verify BVN and NIN | Catch routes to rejection branch + SNS alert |
| 2 | Task | Lambda queries credit bureau. Score stored in execution payload | Retry for transient API failures |
| 3 | Choice | Score ≥ 650 → approve path; < 650 → manual review path | Manual review includes Wait State + Parallel State |
| 4 | Task | Lambda triggers Paystack API to transfer funds | Retry (3 attempts, exponential backoff) + Catch for manual resolution |
| 5 | Parallel | Two branches: (1) SNS sends SMS/push; (2) DynamoDB writes transaction record for CBN | Ends at Succeed State with receipt |

### Discussion Question

If the Paystack API call fails at Step 4 after all retries are exhausted, what should happen? How would you use Retry and Catch to handle this without losing the transaction? Consider idempotency — what if the transfer succeeded but the acknowledgment was lost?

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Standard Workflows are best for high-volume streaming" | ❌ Standard has 2,000 executions/second limit. For "high-throughput," "streaming," or "IoT sensors" → Express Workflows (100,000/sec). |
| "Exactly-once vs at-least-once is just academic" | ❌ Standard = exactly-once (each state executes once). Express = at-least-once (states may retry). Lambda must be idempotent with Express. |
| "Step Functions runs business logic" | ❌ Step Functions orchestrates — it does NOT compute. All business logic lives in Lambda or integrated services. |
| "Activity Tasks are the same as Service Integrations" | ❌ Activity Task = worker polls for work (pull model). Service Integration = direct sync call to Lambda/DynamoDB/SNS (push model). Question says "worker polls" → Activity Task. |

---

## 9. Key Metrics Summary

| Metric | Standard | Express |
|--------|----------|---------|
| Max duration | 1 year | 5 minutes |
| Executions/second | 2,000 | 100,000 |
| Execution model | Exactly-once | At-least-once |
| History retention | 90 days (console) | CloudWatch Logs only |
| Pricing | Per state transition | Per execution + duration |

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| Step Functions purpose | Orchestrate multi-step workflows (not compute) |
| State machine defined in | ASL (Amazon States Language — JSON) |
| Most common state type | Task (calls Lambda or services) |
| Branching/if-else logic | Choice State |
| Pause for human approval | Wait State |
| Run independent tasks together | Parallel State |
| Process array of items | Map State |
| Retry built-in | Yes — configure per Task State |
| Express max duration | 5 minutes |
| Standard max executions/sec | 2,000 |
| Express max executions/sec | 100,000 |
| "Worker polls for work" | Activity Task (pull) vs Service Integration (push) |

---

## 11. One Sentence to Remember

> **Step Functions orchestrates multi-step workflows (not compute). Task = call service, Choice = branch, Wait = pause, Parallel = simultaneous, Map = iterate array. Standard = exactly-once, 1 year, 2,000/sec, per-state pricing, 90-day audit. Express = at-least-once, 5 min, 100,000/sec, per-execution pricing, CloudWatch only. For high-throughput streaming → Express.**