# ☁️ Week 3 · Day 5: VPC Peering & Connectivity

**Module:** Networking & VPC  
**Certification Alignment:** AWS SAA-C03

---

## 1. The Big Picture

Real-world AWS deployments rarely live in a single VPC. Most organizations run separate VPCs for production, development, staging, and different business units. AWS gives you **five tools** to connect them.

---

## 2. The Five Connectivity Tools (Mental Model)

| Tool | Analogy | Best for |
|------|---------|----------|
| **VPC Peering** | Private road between two specific buildings | 2–3 VPCs |
| **Site-to-Site VPN** | Armored van on public highway | On-premises, low volume, quick setup |
| **Direct Connect** | Dedicated private motorway | On-premises, high volume, consistent latency |
| **Transit Gateway** | Central motorway interchange | 10+ VPCs, hub-and-spoke |
| **VPC Endpoints** | Internal office lift to AWS services | Private S3/API access without internet |

---

## 3. VPC Peering (Private, Direct & Non-Transitive)

| Feature | Detail |
|---------|--------|
| **What it is** | Direct private network link between two VPCs |
| **Traffic path** | AWS private backbone (never the internet) |
| **CIDR requirement** | Cannot overlap |
| **Cross-account?** | ✅ Yes |
| **Cross-region?** | ✅ Yes (higher data transfer cost) |

### Critical Rule: Non-Transitive
If VPC A ↔ VPC B and VPC B ↔ VPC C
Then A CANNOT reach C through B


**For 10 VPCs with full peering:** 45 connections → use Transit Gateway instead.

### Route Tables (BOTH VPCs)

Peering alone routes nothing. You must add routes in **both** VPCs' route tables pointing to each other's CIDR. One-way routes = one-way traffic.

---

## 4. Connecting On-Premises to AWS

| | Site-to-Site VPN | Direct Connect |
|--|------------------|----------------|
| **Path** | Public internet (IPsec encrypted) | Dedicated private fiber |
| **Encryption** | ✅ Yes (by default) | ❌ No (add VPN over DX) |
| **Speed** | Up to 1.25 Gbps | 1/10/100 Gbps dedicated |
| **Latency** | Variable | Consistent low |
| **Setup time** | Minutes | Weeks to months |
| **Cost** | ~$0.05/hr | ~$216/month (1Gbps) + data |
| **Best for** | Branch offices, dev, backup | High-volume, compliance, trading |

### CBN Compliance Combo

**Direct Connect + VPN** = private path + encryption. Best of both worlds for regulated environments.

---

## 5. Transit Gateway (Hub-and-Spoke for Scale)

| Feature | Detail |
|---------|--------|
| **What it is** | Regional hub — each VPC connects once |
| **For 10 VPCs** | 10 attachments (vs 45 peering connections) |
| **Route table isolation** | Dev VPC can't reach Prod even through TGW |
| **Attachments** | VPCs, VPNs, Direct Connect Gateways |
| **Cost** | $0.05/hr per attachment + $0.02/GB |

---

## 6. VPC Endpoints (Private Access to AWS Services)

| Type | Services | Cost | How |
|------|----------|------|-----|
| **Gateway Endpoint** | S3, DynamoDB only | **FREE** | Adds route table entry |
| **Interface Endpoint** | 100+ services (SSM, SQS, SNS, etc.) | ~$0.01/hr per AZ + $0.01/GB | Creates ENI with private IP |

**Why Gateway Endpoints are great:** Private subnets can reach S3 without NAT Gateway. Zero internet exposure. Zero cost.

---

## 7. Decision Framework (Exam Cheat Sheet)

| Scenario | Best Tool | Why |
|----------|-----------|-----|
| Connect 2–3 VPCs | **VPC Peering** | Simple, cheap, private |
| Connect 10+ VPCs | **Transit Gateway** | Hub-and-spoke scales cleanly |
| On-prem, low volume, quick setup | **Site-to-Site VPN** | Minutes to deploy, encrypted |
| On-prem, high volume, low latency | **Direct Connect** | Dedicated, consistent, cheap per GB at scale |
| Need encryption + private path (CBN) | **DX + VPN** | Private path + IPsec encryption |
| Private subnet → S3 | **Gateway Endpoint** | FREE, no internet |
| Private subnet → other AWS services | **Interface Endpoint** | PrivateLink, no NAT needed |

---

## 8. Exam Traps (Memorize These)

| Trap | Truth |
|------|-------|
| "Peering is transitive" | ❌ **Non-transitive.** A↔B↔C does NOT mean A can reach C. |
| "Peering alone routes traffic" | ❌ Must add routes in **both** VPCs' route tables. |
| "Direct Connect is encrypted" | ❌ **Not encrypted by default.** Add VPN over DX for encryption. |
| "VPN has one tunnel" | ❌ **Two tunnels.** Monitor both. One fails = partial outage. |
| "Gateway Endpoints cost money" | ❌ **FREE** for S3 and DynamoDB. Always use them. |

---

## 9. Real-World Scenario (Zenith Bank-style)

**Setup:** 3 AWS accounts (Prod, Analytics, Dev) + 2 data centers (Lagos, Abuja)

| Connection | Tool | Why |
|------------|------|-----|
| Prod ↔ Analytics ↔ Shared Services | **Transit Gateway** | Hub-and-spoke. Separate route tables: Dev cannot reach Prod. |
| Lagos DC (500GB+/day, <20ms latency) | **Direct Connect** | High volume, low latency required for treasury |
| Abuja DC (10-50GB/day, no latency req) | **Site-to-Site VPN** | Lower volume, quick setup, ~$36/month |
| Core banking private subnet → S3 audit logs | **Gateway Endpoint** | Zero internet path (CBN requirement), zero cost |

---

## 10. One Sentence to Remember

> **Few VPCs = Peering. Many VPCs = Transit Gateway. On-prem low volume = VPN. On-prem high volume = Direct Connect. Private subnet to S3 = Gateway Endpoint (FREE). Private subnet to other AWS services = Interface Endpoint.**
