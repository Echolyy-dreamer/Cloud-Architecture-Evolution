# Project 1: LAMP Stack Modernization & Cloud-Native Evolution

## 🌟 Executive Summary
This project demonstrates the transition of a legacy **LAMP (Linux, Apache, MySQL, PHP)** monolith
from an on-premises data center to a highly optimized, **2026-standard AWS Cloud-Native architecture**.

Unlike a simple *Lift-and-Shift*, this evolution focuses on:

- **Cost Governance in the IPv6 era**
- **Zero-Trust Security**
- **Serverless & Managed Data Tiering**

The architecture aligns with the expectations of:

- **AWS Certified Solutions Architect – Professional (SAP-C02)**
- **AWS Advanced Networking – Specialty (ANS-C01)**

---

## 🏗️ Architecture Evolution

### 1. Core Business Path (L1 View)

The L1 view highlights the **primary revenue path**:
- Global content delivery
- Static asset offloading
- Read / Write database separation

```mermaid
flowchart TB
    U2["Global User<br/>(Dual-Stack IPv4/IPv6)"]
    CF["CloudFront<br/>+ Origin Shield"]
    ALB["ALB<br/>(L7 Load Balancer)"]
    ASG["ASG<br/>(Graviton4 m8g)"]
    S3["S3<br/>(Static Assets)"]
    Proxy["RDS Proxy"]

    subgraph Aurora ["Aurora Cluster (Multi-AZ)"]
        direction LR
        RW["Primary (RW)"] -.->|Sync Replication| RO["Reader (RO)"]
    end

    U2 --> CF
    CF -- Dynamic --> ALB --> ASG
    CF -- Static --> S3
    ASG -- Media / Uploads --> S3
    ASG --> Proxy
    Proxy --> RW
    Proxy -.-> RO
🛡️ Production-Grade Details (L2 / L3 Deep Dive)
<details> <summary><b>🚀 Click to Expand: Security, Observability & Governance</b></summary>

This diagram exposes the non-obvious enterprise layers required for
production readiness: private connectivity, encryption boundaries,
and request-path observability.
```mermaid
flowchart TB
    subgraph O ["✅ 2026 OPTIMIZED BLUEPRINT"]
        direction TB
        U2["Global User"]

        %% Traffic & Compute
        subgraph Traffic ["Edge & Compute"]
            CF["CloudFront"]
            WAF["AWS WAF"]
            ALB["ALB"]
            ASG["ASG (Graviton4)"]
            U2 --> CF --> WAF --> ALB --> ASG
        end

        %% Data & Storage
        subgraph Data ["Data & Storage"]
            direction LR
            S3["S3 Intelligent-Tiering"]
            Proxy["RDS Proxy"]

            subgraph Aurora ["Aurora Serverless v2"]
                direction LR
                RDSW["Primary"] -.->|Sync| RDSR["Reader"]
            end

            ASG --> Proxy --> Aurora
            ASG -- Media --> S3
            CF -- Cached Assets --> S3
        end

        %% Management & Observability
        subgraph Ops ["Management & Observability"]
            VPCE["VPC Endpoints<br/>(PrivateLink)"]
            CWL["CloudWatch<br/>Logs & Metrics"]
            XRY["X-Ray Traces"]
            KMS["KMS<br/>(Envelope Encryption)"]
            SSM["SSM<br/>(No-SSH Access)"]
        end

        %% Control Paths
        ASG -.-> VPCE
        VPCE -.-> S3
        VPCE -.-> KMS
        ASG -.-> CWL
        ASG -.-> XRY
        S3 --> GLC["Glacier Deep Archive"]
    end
💎 Technical Pillars & Optimization
1. Advanced Networking (ANS-C01 Aligned)

IPv6-First Design
Backend workloads operate in IPv6-only subnets to avoid public IPv4 costs.

Egress-Only Internet Gateway
Enables outbound connectivity without NAT Gateway cost or scaling limits.

VPC Endpoints (PrivateLink)
S3 and KMS traffic never leaves the AWS backbone.

2. Compute & Cost Governance

Graviton4 (m8g)
~40% price-performance improvement for PHP-FPM workloads.

Stateless Compute
Media and static assets offloaded to S3 enable rapid scale-out and instance recycling.

3. Data Integrity & Resilience (SAP-C02 Standard)

RDS Proxy
Mitigates PHP connection storms and reduces failover impact.

S3 Intelligent-Tiering
Automatic cost optimization for wp-content and media assets.

Aurora Global-Ready
Architecture prepared for future cross-region read expansion.
📉 Architecture Decision Records (ADR)
<details> <summary><b>📊 Click to View Technology Selections</b></summary>
Category	Selection	Justification
Compute	Graviton4 (m8g)	Best price-performance for PHP 8.x
Database	Aurora Serverless v2	Elastic scaling for bursty traffic
Security	WAF + Shield	Edge-layer threat mitigation
Operations	SSM Session Manager	Zero open inbound ports
Storage	S3 + CloudFront	Stateless application tier
</details>
🚀 Future Roadmap

 Infrastructure as Code (Terraform)

 Deep PHP query tracing via X-Ray

 Migration path to Amazon EKS (Fargate)
