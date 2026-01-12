# Project 1: LAMP Stack Modernization & Cloud-Native Evolution

## 🌟 Executive Summary

This project demonstrates the modernization of a legacy **LAMP (Linux, Apache, MySQL, PHP)** monolith from an on-premises data center into a production-grade AWS cloud-native architecture (2026-ready).

This is not a lift-and-shift exercise. The focus is on:

* **IPv6-first cost governance** (Eliminating IPv4 address tax)
* **Zero-trust & Private connectivity** (AWS PrivateLink & WAF)
* **Stateless compute** (Graviton4) and managed serverless data tiers

---

## 🏗️ Architecture Evolution

### 1️⃣ Core Business Path — L1 (The Revenue Flow)

This view highlights the revenue-critical request path. All operational and governance layers are hidden to focus on high-speed content delivery and data persistence.

```mermaid
flowchart TB
    U["Global User (Dual-Stack)"]
    CF["CloudFront Origin Shield"]
    ALB["Application Load Balancer"]
    ASG["EC2 ASG Graviton4 (m8g)"]
    S3["S3 Static & Media"]
    RP["RDS Proxy"]

    subgraph DB["Aurora MySQL (Multi-AZ)"]
        direction LR
        RW["Writer"]
        RO["Reader"]
    end

    U --> CF
    CF -- "Dynamic Content" --> ALB --> ASG
    CF -- "Static Assets" --> S3
    ASG -- "Uploads" --> S3
    ASG --> RP --> RW
    RP -.->|Read Scaling| RO
```

🛡️ **Production-Grade Details (L2/L3 Enterprise Deep Dive)**

<details>
<summary><b>Click to Expand: Enterprise Security & Observability Layer</b></summary>

This diagram exposes the non-obvious enterprise layers required for production readiness: private connectivity, encryption boundaries, and request-path observability.

```mermaid
flowchart TB
    %% GLOBAL
    U["Global User"]

    %% EDGE & COMPUTE
    subgraph Edge["Edge & Compute Layer"]
        CF["CloudFront"]
        WAF["AWS WAF"]
        ALB["ALB"]
        ASG["EC2 ASG Graviton4"]
        U --> CF --> WAF --> ALB --> ASG
    end

    %% DATA
    subgraph Data["Data Layer"]
        S3["S3 Intelligent-Tiering"]
        RP["RDS Proxy"]

        subgraph Aurora["Aurora Serverless v2"]
            direction LR
            RW["Writer"] -.->|Sync Replication| RO["Reader"]
        end

        ASG --> RP --> Aurora
        ASG -->|Media Offloading| S3
        CF -->|Cached Assets| S3
    end

    %% OBSERVABILITY & CONTROL
    subgraph Ops["Observability & Control"]
        CW["CloudWatch"]
        XR["X-Ray"]
        KMS["KMS (Envelope Encryption)"]
        VPCE["VPC Endpoints"]
        SSM["SSM Session Manager"]
    end

    %% CONTROL FLOWS
    ASG -.-> CW
    ASG -.-> XR
    ASG -.-> VPCE
    VPCE -.-> S3
    VPCE -.-> KMS
    S3 --> GA["Glacier Deep Archive"]
```

</details>

---

## 💎 Technical Pillars (Architectural Reasoning)

### 1️⃣ Advanced Networking 

* IPv6-First Subnets: Eliminates NAT Gateway dependency and avoids the rising costs of Public IPv4 addresses.
* PrivateLink (VPC Endpoints): S3 and KMS traffic never leaves the AWS backbone, ensuring deterministic latency and a hard security boundary.

### 2️⃣ Compute & Cost Governance

* Graviton4 (m8g): Achieves ~40% better price-performance for PHP-FPM workloads compared to x86 equivalents.
* Stateless Tiering: All mutable state is offloaded to S3 or Aurora, allowing the ASG to scale aggressively and instances to be recycled without data loss.

### 3️⃣ Data Resilience 

* RDS Proxy: Absorbs connection "storms" typical of PHP applications and enables seamless failover by reducing recovery time by up to 60%.
* Aurora Serverless v2: Provides elastic scaling (0.5 to 128 ACUs) to match unpredictable traffic while maintaining High Availability.
* S3 Intelligent-Tiering: Automatic lifecycle management to move aged media to cheaper storage tiers (Glacier) without application changes.

---

## 📊 Architecture Decision Records (ADR)

| Area     | Decision             | Reasoning                                                               |
| -------- | -------------------- | ----------------------------------------------------------------------- |
| Compute  | EC2 Graviton4 ASG    | Optimized for PHP JIT performance; significantly lower TCO than R7i     |
| Database | Aurora Serverless v2 | Handles bursty traffic; sub-minute failover via RDS Proxy               |
| Security | WAF + PrivateLink    | Edge-layer filtering combined with Zero-Trust internal connectivity     |
| Ops      | SSM (No SSH)         | Eliminates Bastion Hosts; audit-trailed access via IAM                  |
| Storage  | S3 + CloudFront      | Decoupling static assets from the compute layer to enable stateless EC2 |

---

## 🚀 Future Roadmap

* [ ] Infrastructure as Code: Full deployment via Terraform/CDK.
* [ ] Observability: Implementing deep PHP tracing with X-Ray subsegments.
* [ ] Containerization: Migration path to EKS on Fargate for microservices.
* [ ] Global Scale: Implementation of Aurora Global Database for sub-second cross-region DR.
