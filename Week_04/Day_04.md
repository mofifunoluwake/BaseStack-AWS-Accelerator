# ☁️ Week 4 · Day 4: Auto Scaling & Load Balancing

**Module:** Compute  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Static capacity creates two problems — both wrong:

| Approach | Problem |
|----------|---------|
| **Provision for peak** | You pay for 20 servers when you only need 5 most of the time. Wasted money. |
| **Provision for average** | Black Friday kills your servers. Lost customers. |

**Auto Scaling + Load Balancing = Elasticity** — start with 3 instances, scale to 20+ during spikes, scale back down. Pay only for what you use.

---

## 2. The Nigerian Restaurant Analogy

| Scenario | What happens |
|----------|--------------|
| Normal day | 3 workers enough |
| Lunch rush (no extra workers) | Customers wait too long, some leave |
| Hiring 20 workers every day | Waste money during quiet hours |
| **Auto Scaling** | Bring in more workers only when crowd increases |
| **Load Balancing** | Manager shares customers across all workers so nobody is overloaded |

---

## 3. Auto Scaling Group (ASG) — Core Components

An ASG is not a single setting — it's an orchestrated system.

### Launch Template — The Blueprint

Defines what to launch: AMI, instance type, key pair, security groups, user data script. Every new instance comes out identical. **Always use Launch Templates** (Launch Configurations are deprecated).

### Capacity Settings — Min / Desired / Max

| Setting | What it does |
|---------|--------------|
| **Min** | Floor — ASG never goes below this |
| **Desired** | Current target — ASG converges here |
| **Max** | Ceiling — hard cap on cost |

**Example:** Min=2, Desired=4, Max=10

### Scaling Policies — When & How to Scale

| Policy | How it works | Best for |
|--------|--------------|----------|
| **Target Tracking** | "Keep CPU at 60%" — AWS handles the math | Web apps (simplest) |
| **Step Scaling** | "If CPU > 80%, add 3 instances" | Complex workloads with multiple thresholds |
| **Scheduled Scaling** | "Every Friday 9am, scale to 20" | Predictable, recurring spikes (payday, market open) |
| **Predictive Scaling** | ML analyzes past patterns, pre-launches capacity before demand hits | Recurring patterns (salary day in fintech) |

### Health Checks

| Type | What it detects |
|------|-----------------|
| **EC2 checks** | Hardware/hypervisor failure |
| **ELB checks** | App responds with HTTP 200 |

**Failed instances are auto-terminated and replaced** — automatic self-healing.

---

## 4. Elastic Load Balancing (ELB) — Three Types

A Load Balancer sits in front of EC2 instances, routes requests across them, performs health checks, and stops sending traffic to unhealthy instances.

### ALB — Application Load Balancer (Layer 7)

| Attribute | Detail |
|-----------|--------|
| **Layer** | 7 (HTTP/HTTPS) |
| **Best for** | Web apps, microservices, containers |
| **Features** | Path-based routing (`/api` vs `/static`), host-based routing, WebSocket, HTTP/2, WAF integration, sticky sessions, TLS termination |
| **Exam clue** | "path-based routing", "microservices", "HTTP/HTTPS" → ALB |

### NLB — Network Load Balancer (Layer 4)

| Attribute | Detail |
|-----------|--------|
| **Layer** | 4 (TCP/UDP/TLS) |
| **Best for** | Ultra-low latency, gaming, IoT, fintech |
| **Features** | Millions of requests/sec, static IP, preserves source IP, TLS passthrough |
| **Exam clue** | "static IP", "millions of req/sec", "preserve source IP", "TCP/UDP" → NLB |

### GWLB — Gateway Load Balancer (Layer 3)

| Attribute | Detail |
|-----------|--------|
| **Layer** | 3 (IP packets) |
| **Best for** | Third-party security appliances, deep packet inspection |
| **Features** | Deploys/scales virtual appliances, VPC traffic mirroring, firewalls, IDS/IPS |
| **Exam clue** | "firewall", "IDS/IPS", "security appliance", "inspect all traffic" → GWLB |

---

## 5. ASG + ELB Together — The Architecture Flow

Client Request → Route 53 (DNS) → ALB (Load Balancer) → Healthy EC2 instances

↓

CloudWatch monitors metrics → ASG scales when needed

### Key Integration Points

| Feature | How it works |
|---------|--------------|
| Target groups | ALB target groups register ASG instances automatically |
| Health checks | ASG uses ALB health checks to determine instance health |
| Scaling triggers | CloudWatch alarms trigger ASG scaling policies |
| Failover | Route 53 can health-check the ALB endpoint |
| Self-healing | Failed instances terminated and replaced within ~2 minutes |

### What Happens During a Traffic Spike

1. CloudWatch detects CPU breaching threshold (e.g., 60%)
2. ASG launches new EC2 instances from Launch Template
3. New instances run user data scripts (pending state)
4. ALB health check confirms HTTP 200 → instance moves to **InService**
5. ALB begins routing traffic to new instance automatically
6. When demand falls, ASG scales in — draining connections first

---

## 6. ASG Lifecycle — Scale-Out & Scale-In

### Scale-Out (Adding Instances)

| Step | What happens | Time |
|------|--------------|------|
| 1 | CloudWatch alarm fires (CPU > threshold) | Immediate |
| 2 | EC2 launches instance from Launch Template | ~1-2 min |
| 3 | Instance runs user data, passes health checks | Variable |
| 4 | ALB starts routing traffic to new instance | After health check |
| 5 | Cooldown (default 300s) prevents rapid re-scaling | — |

### Scale-In (Removing Instances)

| Step | What happens |
|------|--------------|
| 1 | Metric drops below threshold after cooldown |
| 2 | Termination policy determines which instance to remove first |
| 3 | Connection draining (default 300s) completes in-flight requests |
| 4 | Instance terminates |

### Termination Policies (Exam Critical)

| Policy | Behavior |
|--------|----------|
| **Default** | Oldest instance with oldest launch config |
| **Custom** | Newest, oldest, closest to next billing hour |

**Exam trap:** Deregistering an instance from ALB does **NOT** immediately terminate traffic. Connection draining (default 300s) allows in-flight requests to complete.

---

## 7. Real-World Scenario (Paystack Nigeria)

**Problem:** Month-end salary day — traffic spikes from 3,000 req/min to 45,000 req/min (15× increase). Fixed 5 instances cause 503 errors and 8-second page loads.

### The Auto Scaling Solution

| Time | Action | Instances |
|------|--------|-----------|
| 6:30am | Predictive Scaling (trained on 6 months data) pre-launches capacity | 5 → 13 |
| 7:00–9:00am | Target Tracking (60% CPU) triggers further scale-out. ALB distributes 45K req/min | 13 → 18 |
| 9:00am–12:00pm | Traffic stabilizes. ALB health checks all green. Response times <180ms | 18 |
| 12:00–1:00pm | Target Tracking scales in gradually. ALB drains connections before termination | 18 → 5 |

### Cost Comparison

| | Cost |
|--|------|
| **With Auto Scaling (event only)** | ₦4,200 |
| **Without ASG (18 instances 24/7)** | ₦85,000/day |
| **Savings in one day** | **₦80,800+** |

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Auto Scaling guarantees zero downtime" | ❌ Scale-out takes **1-2 minutes**. During that window, existing instances may be at reduced capacity. Use Predictive Scaling + pre-warming or keep Min capacity high. |
| "EC2 Auto Scaling and Application Auto Scaling are the same" | ❌ **Different services.** EC2 ASG = EC2 instances. **Application Auto Scaling** = ECS tasks, DynamoDB throughput, Lambda concurrency, Aurora replicas. |
| "ALB and NLB are interchangeable" | ❌ **Never interchangeable on exam.** ALB = Layer 7 (HTTP-aware: path routing, WAF). NLB = Layer 4 (faster, static IP, preserves source IP). |
| "Deregistering from ALB immediately stops traffic" | ❌ ALB has **connection draining** (default 300s). In-flight requests complete before traffic stops. Set to 0 only for stateless, fast-response apps. |

---

## 9. Practice Question (SAA-C03 Style)

**Scenario:** A Lagos fintech runs a payment API on EC2. During month-end salary processing, traffic increases from 500 req/min to 12,000 req/min over 20 minutes. API is stateless. Need automatic handling, response times <200ms, and minimal cost during off-peak.

**Which combination is correct?**

| Option | Answer |
|--------|--------|
| **ALB + ASG with Target Tracking (60% CPU), Min=2, Desired=4, Max=20, Predictive Scaling enabled** | ✅ **Correct.** ALB for HTTP API. Target Tracking simplest. Predictive Scaling handles predictable spike. Min/Max control cost. |
| NLB + Fixed fleet of 20 instances | ❌ Runs 20 instances 24/7 — ignores cost optimization. NLB wrong for HTTP API. |
| Scheduled Scaling only | ❌ Misses irregular spikes outside scheduled window. |
| Step Scaling + 600s cooldown | ❌ 600s cooldown too long for 20-minute spike window. |

---

## 10. Quick Reference Card

| Question | Answer |
|----------|--------|
| ALB operates at which layer? | Layer 7 (HTTP/HTTPS) |
| NLB operates at which layer? | Layer 4 (TCP/UDP) |
| GWLB operates at which layer? | Layer 3 (IP packets) |
| When to use ALB? | Path-based routing, microservices, WAF |
| When to use NLB? | Static IP, preserve source IP, ultra-low latency |
| When to use GWLB? | Firewalls, IDS/IPS, security appliances |
| Best scaling policy for most web apps? | Target Tracking |
| Best for predictable recurring spikes? | Scheduled Scaling or Predictive Scaling |
| Default connection draining time? | 300 seconds |
| What detects hardware failure? | EC2 health checks |
| What detects app returning 500 errors? | ELB health checks |

---

## 11. One Sentence to Remember

> **ASG = Launch Template + Min/Desired/Max + scaling policy. ALB for Layer 7 (path routing, WAF), NLB for Layer 4 (static IP, speed, source IP preservation), GWLB for security appliances. Target Tracking is default scaling policy. Scale-out takes 1-2 minutes. Connection draining protects in-flight requests (default 300s). Failed instances auto-replace — that's self-healing.**
