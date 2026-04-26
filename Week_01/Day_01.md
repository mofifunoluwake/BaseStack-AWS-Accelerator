# ☁️ Day 1: What Is Cloud Computing?
**Module:** Cloud Foundations 
**Certification Alignment:** AWS SAA-C03 

## 1. The Core Paradigm Shift
Cloud computing democratises enterprise-grade infrastructure. It replaces the traditional model of buying physical servers (Capital Expenditure / CapEx) with renting resources on-demand (Operational Expenditure / OpEx). 

**Key Advantages:** * Stop guessing capacity.
* Benefit from massive economies of scale.
* Increase speed and agility.
* Stop spending money running and maintaining data centres.
* Go global in minutes.

## 2. The 5 Essential Characteristics (NIST Definition)
* **On-Demand Self-Service:** Provision resources instantly without human interaction.
* **Broad Network Access:** Accessible from anywhere via standard internet.
* **Resource Pooling:** Multiple tenants securely share the same physical infrastructure via multi-tenancy.
* **Rapid Elasticity:** Automatically scale up during peak demand and scale down to save costs.
* **Measured Service:** Pay-as-you-go pricing, functioning exactly like a utility bill.

## 3. The Architecture: How It Works
AWS maintains massive data centres. Through virtualisation, physical hardware is sliced into Virtual Machines (VMs), such as Amazon EC2 instances, which users rent.

### Service Models (The Responsibility Stack)
* **IaaS (Infrastructure as a Service):** Rent raw computing resources. You manage the OS, security, and applications.
* **PaaS (Platform as a Service):** AWS manages the underlying servers and OS. You just bring your application code.
* **SaaS (Software as a Service):** Fully finished applications (e.g., Gmail, Dropbox). Zero infrastructure management.

### Deployment Models
* **Public:** Shared infrastructure, managed by AWS.
* **Private:** Dedicated solely to one organisation.
* **Hybrid:** A mix of on-premises infrastructure and cloud resources.
* **Multi-Cloud:** Utilising multiple different cloud providers simultaneously.

## 4. Critical Exam Traps (SAA-C03)
* Cloud computing has exactly **FIVE** essential NIST characteristics, not three or four.
* Understand the boundary lines between IaaS, PaaS, and SaaS (who manages what layer).
* **On-Demand** pricing requires absolutely NO upfront commitment (unlike Reserved Instances or Savings Plans).

## 5. Real-World Context: Nigerian Fintech Scenario
A mobile payment startup in Lagos no longer needs to spend ₦15 million upfront to buy physical servers and rent data centre space. Using AWS, their infrastructure automatically scales up to handle massive traffic spikes (like a holiday promo) and scales down during quiet hours. They never pay for idle hardware.
