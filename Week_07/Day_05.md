# ☁️ Week 7 · Day 5: CloudWatch & Observability

**Module:** Monitoring & Observability  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture — You Can't Fix What You Can't See

Deploying to AWS is step one. The real engineering work starts when something breaks at 2am and you need to know exactly why — fast.

**Observability** is the practice of understanding a system's internal state by examining the signals it emits: logs, metrics, and traces. CloudWatch is AWS's native platform for all three.

| Without CloudWatch | With CloudWatch |
|--------------------|-----------------|
| Manual SSH debugging | Automated alerting |
| Hours of downtime | Minutes to detect and fix |
| Customers complain on social media | Team knows before users notice |
| Revenue loss and reputational damage | Proactive incident response |

---

## 2. The Three Pillars of CloudWatch

| Pillar | What It Is | What It Answers | Example |
|--------|------------|-----------------|---------|
| **Metrics** | Numerical measurements over time | "How much?" | CPU 75%, error rate 5%, latency 200ms |
| **Logs** | Text records of events | "What happened?" | "ERROR: DB connection pool exhausted" |
| **Alarms** | Rules that watch metrics and fire actions | "When to alert?" | If CPU > 80% for 5 min → page on-call |

### Metrics

| Attribute | Detail |
|-----------|--------|
| **Resolution** | Standard = 1 minute; High-resolution custom = 1 second (extra cost) |
| **Retention** | 15 months by default |
| **Source** | Every AWS service publishes metrics automatically |

### Logs

| Attribute | Detail |
|-----------|--------|
| **Retention** | 1 day – 10 years (configurable per Log Group) |
| **Organization** | Log Groups (by service) → Log Streams (per instance/function) |
| **Delivery** | Near real-time |

### Alarms

| Attribute | Detail |
|-----------|--------|
| **States** | OK (within threshold), ALARM (breached), INSUFFICIENT_DATA (not enough data) |
| **Actions** | SNS notification, Auto Scaling policy, EC2 stop/reboot, Lambda |
| **Auto-resolve** | Returns to OK automatically when metric drops below threshold |

---

## 3. Advanced CloudWatch Features

### CloudWatch Dashboards

| Attribute | Detail |
|-----------|--------|
| **What** | Customizable visual panels displaying metrics and alarms |
| **Scope** | Global (cross-region) — view multiple services in one place |
| **Free tier** | Up to 50 widgets per dashboard |

### CloudWatch Logs Insights

| Attribute | Detail |
|-----------|--------|
| **What** | Powerful query engine for logs |
| **Query limit** | Up to 10,000 records per query |
| **Example query** | `fields @timestamp, @message \| filter @message like /ERROR/ \| sort @timestamp desc \| limit 50` |

### CloudWatch Synthetics (Canaries)

| Attribute | Detail |
|-----------|--------|
| **What** | Scripted monitors that simulate real user actions (click a button, complete checkout) |
| **Schedule** | Once per minute to once per day |
| **Exam trap** | Synthetics actively performs user journeys — NOT the same as standard alarms |

---

## 4. How the CloudWatch Alert Pipeline Works

```text
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Service   │ ──→ │   Logs/     │ ──→ │   Alarm     │ ──→ │   Action    │
│   Emits     │     │   Metrics   │     │   Watches   │     │   Fires     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘

Example: Lambda error rate > 5% for 3 minutes → SNS → Slack + phone
```
| **Stage** | **WHat happens** |
|-----------|--------|
| 1 | Service emits metrics and logs (Lambda, EC2, API Gateway, etc.) |
| 2 | CloudWatch collects and stores data |
| 3 | Alarm evaluates metric against threshold (e.g., CPU > 80% for 5 min) |
| 4 | Alarm triggers action (SNS, Auto Scaling, Lambda, EC2 action) |

## 5. Important Numbers & Limits

| Setting | Value |
|---------|-------|
| Default metric retention | 15 months |
| Log retention (configurable) | 1 day – 10 years |
| Alarm evaluation periods | Up to 1,440 datapoints |
| Dashboard widgets (free) | Up to 50 per dashboard |
| Custom metrics (free tier) | 10 metrics per month |
| Logs Insights query result | Up to 10,000 records |

---

## 6. The CloudWatch Agent — Why It Matters (Exam Critical)

EC2 instances do **NOT** send memory or disk metrics by default.

| Metric | Collected by default? |
|--------|----------------------|
| CPU utilization | ✅ Yes |
| Network | ✅ Yes |
| Disk I/O | ✅ Yes |
| Memory utilization | ❌ No — requires CloudWatch Agent |
| Disk space | ❌ No — requires CloudWatch Agent |

**Installation workflow:** Use SSM Parameter Store to push agent configuration → agent collects metrics automatically → data ships to CloudWatch.

**Exam trap:** If a question mentions EC2 memory alarms not firing, the answer is almost always "CloudWatch Agent is not installed." This is the single most tested CloudWatch trap on the SAA-C03.

---

## 7. Real-World Nigerian Scenario — OluPay's Friday Evening Crisis

**Problem:** Payment API suddenly has spike in failed transfers on Friday evening (busiest time). Without observability → 2+ hours of downtime, customers complaining on Twitter.

### With CloudWatch

| Step | Time | Action |
|------|------|--------|
| 1 | < 1 min | Alarm fires: Lambda error rate > 5% for 3 minutes → SNS to Slack + engineer's phone |
| 2 | < 2 min | Dashboard shows API latency spiked at 6:47pm — pinpointing exact moment |
| 3 | < 5 min | Logs Insights query finds all ERROR logs in last hour |
| 4 | 8 min | Root cause found: DB connection pool exhausted → fix deployed |
| 5 | Auto | Alarm returns to OK automatically → team notified |

### Without CloudWatch

| Problem | Impact |
|---------|--------|
| Manual SSH debugging | 2+ hours of downtime |
| Scrolling raw logs | No centralized view |
| Customers complain on social media | Revenue loss + reputational damage |

**Result:** Incident detected in <1 minute. Root cause identified in 8 minutes. Zero manual SSH needed.

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "CloudWatch automatically collects EC2 memory metrics" | ❌ Only CPU, disk I/O, and network are default. Memory and disk require CloudWatch Agent. Most tested trap. |
| "CloudWatch is for auditing who deleted an S3 bucket" | ❌ CloudWatch = performance (metrics, logs, alarms). CloudTrail = audit (who did what, when, from where). |
| "Alarms stay in ALARM state until manually reset" | ❌ Alarms auto-resolve — return to OK when metric drops below threshold. No manual reset needed. |
| "Standard metrics have 1-second resolution" | ❌ Standard = 1 minute. High-resolution custom = 1 second (costs extra). |
| "Synthetics is the same as standard alarms" | ❌ Synthetics = canaries (simulate user actions). Standard alarms = passive metric watching. |

---

## 9. CloudWatch vs CloudTrail — Know the Difference

| Question | CloudWatch | CloudTrail |
|----------|------------|------------|
| What does it monitor? | Performance — how the system is behaving | API calls — who did what, when, from where |
| "Is my CPU high?" | ✅ Answers this | ❌ |
| "Who deleted my S3 bucket?" | ❌ | ✅ Answers this |
| "Are my Lambda errors spiking?" | ✅ Answers this | ❌ |
| "Did someone change an IAM policy?" | ❌ | ✅ Answers this |
| Automated alerting | ✅ Alarms + SNS | ❌ |

**Exam shortcut:** Performance monitoring → CloudWatch. Compliance/audit → CloudTrail.

---

## 10. Practice Question (SAA-C03 Style)

**Scenario:** A DevOps team notices that their EC2-based web server is running out of memory but no CloudWatch alarm has fired. The EC2 CPUUtilization alarm is configured correctly.

**What is the MOST likely reason?**

| Option | Answer |
|--------|--------|
| The alarm threshold is set too high | ❌ Alarm is for CPU, not memory — irrelevant |
| **CloudWatch Agent is not installed** | ✅ **Correct** — EC2 does not publish memory metrics by default. Without CloudWatch Agent, no memory data exists for alarm to evaluate. |
| Memory alarms require a separate IAM role | ❌ IAM role is not the issue |
| EC2 must be in a VPC to publish metrics | ❌ EC2 publishes metrics regardless of VPC |

**Answer:** B — Without the CloudWatch Agent installed, there is no memory data for an alarm to evaluate, so no alarm can ever fire.

---

## 11. Quick Reference Card

| Question | Answer |
|----------|--------|
| CloudWatch default metric retention | 15 months |
| EC2 metrics collected by default | CPU, network, disk I/O only |
| EC2 memory/disk metrics require | CloudWatch Agent |
| Alarm states | OK, ALARM, INSUFFICIENT_DATA |
| Do alarms auto-resolve? | Yes — when metric drops below threshold |
| Logs Insights query limit | 10,000 records |
| Synthetics (Canaries) simulate | Real user actions |
| CloudWatch = | Performance monitoring |
| CloudTrail = | API audit (who did what) |

---

## 12. One Sentence to Remember

> **CloudWatch = metrics (15 months retention), logs (1 day–10 years), alarms (OK/ALARM/INSUFFICIENT_DATA, auto-resolve). EC2 memory/disk metrics require CloudWatch Agent (not automatic). CloudWatch = performance. CloudTrail = audit (who did what). Synthetics canaries simulate user actions (not standard alarms).**