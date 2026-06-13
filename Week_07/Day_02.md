# ☁️ Week 7 · Day 2: API Gateway & Event-Driven Architecture

**Module:** Serverless  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — What Is Amazon API Gateway?

Think of API Gateway like a bank teller. You don't walk into the bank vault directly — a teller verifies your identity and fetches what you need. API Gateway stands between the internet and your Lambda functions.

| Feature | Benefit |
|---------|---------|
| **Fully Managed** | AWS handles scaling, patching, and availability automatically |
| **Secure by Design** | HTTPS, auth tokens, and rate limiting built in |
| **Lambda's Front Door** | The only way to expose Lambda as a public HTTPS API endpoint |

**SAA-C03 Exam Tip:** API Gateway is the only way to expose Lambda as a public HTTPS API endpoint. This is a frequently tested concept.

---

## 2. Types of APIs in AWS API Gateway

| API Type | Best for | Cost | Key Features |
|----------|----------|------|--------------|
| **REST API** | Complex enterprise APIs | More expensive | Caching, API keys, request/response transforms |
| **HTTP API** | Lambda integrations (default choice) | **71% cheaper** | Simpler, faster, best for Lambda |
| **WebSocket API** | Real-time bidirectional apps | Variable | Persistent two-way connection, chat, live dashboards |

**Examples:**
- REST API → Konga product catalogue API
- HTTP API → PalmPay mobile payment API
- WebSocket API → Flutterwave transaction alerts

---

## 3. The Request-Response Flow

| Step | Component | Action |
|------|-----------|--------|
| 1 | Mobile App | Sends HTTPS request to API Gateway URL |
| 2 | API Gateway | Validates auth tokens, API keys, rate limits, request format |
| 3 | Lambda | Gateway triggers Lambda function with request data |
| 4 | Lambda | Code runs: reads database, applies business logic |
| 5 | API Gateway | Gateway transforms response and sends JSON back to client |

**⏱ Full round trip — from client request to JSON response — typically under 100 milliseconds.**

---

## 4. Security & Access Control

| Auth Type | How it works | Best for | Exam Tip |
|-----------|--------------|----------|----------|
| **IAM Auth** | Caller signs requests with AWS credentials | Machine-to-machine (EC2 → internal API) | — |
| **API Keys** | Alphanumeric strings identify clients | Rate limiting via Usage Plans | **NOT authentication** — just identification |
| **Cognito Auth** | Cognito issues JWT tokens; Gateway validates automatically | User-facing apps (fintech, school portals) | No custom code needed |
| **Lambda Authorizer** | Lambda inspects request, returns Allow or Deny | Non-standard auth or external OAuth | Most flexible |

**Exam Trap:** API Keys enable rate limiting — they are **NOT authentication**. For real user auth, use Cognito or Lambda Authorizers.

---

## 5. Event-Driven Architecture — Core Components

**Core idea:** Services drop messages onto a queue or topic and move on. The receiver processes when ready. Decouples systems, improves resilience, makes scaling easier.

### Amazon SQS (Simple Queue Service) — 1:1 Queue

| Feature | Detail |
|---------|--------|
| **Pattern** | 1:1 — one consumer per message |
| **Guarantee** | Every message processed at least once |
| **Analogy** | WhatsApp message waiting until you're online |
| **Use for** | Email jobs, reports, payment confirmations |

### Amazon SNS (Simple Notification Service) — 1:Many Fan-Out

| Feature | Detail |
|---------|--------|
| **Pattern** | 1:many — one event, multiple subscribers |
| **Subscribers** | Lambda, SQS, email, SMS, HTTP endpoints |
| **Analogy** | Broadcasting on a WhatsApp channel |
| **Use for** | When many services must react to one event |

### Amazon EventBridge — Smart Routing by Content

| Feature | Detail |
|---------|--------|
| **Pattern** | Routes events by content rules (pattern matching) |
| **Connects** | AWS services + third-party SaaS tools |
| **Analogy** | A switchboard routing calls by topic |
| **Use for** | Complex multi-service orchestration |

### Comparison Table

| Service | Pattern | Use Case |
|---------|---------|----------|
| **SQS** | 1:1 queue | Decouple producers from consumers |
| **SNS** | 1:many fan-out | Broadcast to multiple subscribers |
| **EventBridge** | Content-based routing | Complex event orchestration |

---

## 6. Architecture: API Gateway + Event-Driven

A production payment system combines synchronous request-response (fast path) with asynchronous event-driven (background processing).

### Sync Path (<100ms) — Immediate Response

| Step | Component | Action |
|------|-----------|--------|
| 1 | Mobile App | Sends POST /payments request |
| 2 | API Gateway | Authenticates and routes request |
| 3 | Lambda A | Processes payment, writes to DynamoDB |
| 4 | API Gateway | Returns immediate confirmation to user |

### Async Path (Background) — Non-Blocking

| Step | Component | Action |
|------|-----------|--------|
| 1 | Lambda A | Drops event to SQS queue (non-blocking) |
| 2 | SQS | Holds message until consumer ready |
| 3 | Lambda B | Picks message from queue |
| 4 | SNS | Triggers notification |
| 5 | SMS | User receives receipt alert |

### Flow Summary

| Path | Flow | Purpose |
|------|------|---------|
| **Sync Path (<100ms)** | API Gateway → Lambda A → DynamoDB | Immediate payment confirmation |
| **Async Path (Background)** | Lambda A → SQS → Lambda B → SNS → SMS | SMS alerts, analytics, audit logs — non-blocking |

---

## 7. Real-World Nigerian Scenario — OluPay Fintech

**Context:** Nigerian fintech processing 500,000 daily mobile payments. Need high traffic handling, user auth, real-time SMS alerts, and never down — on a startup budget.

### Key Metrics

| Metric | Value |
|--------|-------|
| API timeout limit | **29 seconds** (Gateway cuts connection) |
| Cost saved (HTTP API vs REST) | **71%** |
| Default throughput | 10,000 req/sec (auto-scales) |
| Uptime SLA | 99.9% (API Gateway managed) |

### How It Works

| Step | Action |
|------|--------|
| 1 | Mobile app sends POST /payments to API Gateway HTTPS endpoint |
| 2 | API Gateway validates JWT from AWS Cognito (user must be logged in) |
| 3 | Gateway invokes Lambda A — processes payment, writes to DynamoDB |
| 4 | Lambda A drops event to SQS queue asynchronously (non-blocking) |
| 5 | Lambda B picks from SQS, triggers SNS, SMS sent via Termii API |

**Discussion:** If OluPay used one server instead, what happens when 50,000 users open the app simultaneously during a Black Friday promo?

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Lambda max timeout is 15 min, so my API can run long tasks" | ❌ API Gateway cuts connection at **29 seconds**. For long jobs: API Gateway → Lambda → SQS → Another Lambda (async). |
| "REST API and HTTP API are the same thing in AWS" | ❌ **HTTP API is 71% cheaper** and best for Lambda. REST API has extra features (caching, API keys, transforms). |
| "SQS and SNS do the same job" | ❌ **SQS = 1:1 queue** (one consumer). **SNS = 1:many fan-out** (many subscribers). Very different. |
| "API Keys protect my API from unauthorized users" | ❌ **API Keys identify clients and enable throttling** — they are NOT authentication. For user auth → Cognito or Lambda Authorizers. |

---

## 9. Practice Question (SAA-C03 Style)

**Scenario:** A Lagos fintech startup needs a payment API: low cost, simple Lambda integration, user auth via their existing Cognito user pool, and no API key management.

**What is the correct approach?**

| Option | Answer |
|--------|--------|
| REST API + Lambda Authorizer + SQS | ❌ REST API costlier; SQS adds latency for sync payments |
| **HTTP API + Cognito Authorizer + Lambda** | ✅ **Correct** — HTTP API = cheap + simple. Cognito = user pool auth. |
| REST API + API Key auth + Direct DynamoDB | ❌ API Keys don't authenticate end users |
| HTTP API + IAM auth + Step Functions | ❌ IAM auth is for services, not end users; Step Functions = overkill |

**Answer:** B — HTTP API is 71% cheaper, Cognito provides user authentication, Lambda runs business logic.

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| API Gateway timeout limit | **29 seconds** |
| Best API type for Lambda | **HTTP API** (71% cheaper than REST) |
| REST API extra features | Caching, API keys, request/response transforms |
| WebSocket API for | Real-time bidirectional (chat, dashboards) |
| SQS pattern | 1:1 queue — one consumer per message |
| SNS pattern | 1:many fan-out — many subscribers per message |
| API Keys provide | Client identification + rate limiting (NOT authentication) |
| User authentication for APIs | Cognito Auth or Lambda Authorizer |
| Machine-to-machine auth | IAM Auth |
| EventBridge does | Routes events by content rules (pattern matching) |

---

## 11. One Sentence to Remember

> **API Gateway is Lambda's front door (required for HTTPS). HTTP API = 71% cheaper, best for Lambda. REST API = caching + API keys. WebSocket = real-time. Gateway timeout = 29 seconds (async long jobs via SQS). SQS = 1:1 queue. SNS = 1:many fan-out. EventBridge = content-based routing. API Keys = rate limiting (NOT auth). User auth = Cognito or Lambda Authorizer.**