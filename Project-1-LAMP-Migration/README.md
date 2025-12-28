
## Overview

This design demonstrates how a small startup can evolve from a single on-premises LAMP server into a resilient, scalable, and AWS-native architecture — without over-provisioning and while preserving operational simplicity.

Rather than assuming known traffic patterns or steady growth, this architecture is intentionally designed for **uncertain and non-linear scaling**, allowing the system to grow naturally as the business evolves.

---

### Architecture Diagram

```mermaid
flowchart LR
  %% =========================
  %% Traditional Architecture
  %% =========================
  subgraph T["❌ Traditional Three-Tier Web Architecture"]
    U1["Users<br/>(Global Access)"]
    LB1["Single Load Balancer<br/>(or None)"]
    WEB1["Web Layer<br/>(Apache / PHP)"]
    APP1["Application Logic<br/>(Tightly Coupled)"]
    DB1["Single Database<br/>(MySQL)"]
    FS1["Local File Storage"]

    U1 --> LB1 --> WEB1 --> APP1 --> DB1
    APP1 --> FS1
  end

  %% =========================
  %% Optimized Architecture
  %% =========================
  subgraph O["✅ Optimized AWS-Native Architecture"]
    U2["Users<br/>(Global)"]

    CDN["CloudFront<br/>(Static Acceleration)"]
    WAF["AWS WAF + Shield"]
    ALB["ALB / NLB"]

    ASG["Auto Scaling Group<br/>(Web / App Tier)"]
    APP2["Stateless Application<br/>(EC2 / Containers)"]

    CACHE["Cache Layer<br/>(ElastiCache / Redis)"]

    RDSW["RDS Writer"]
    RDSR["RDS Read Replica"]

    S3["Amazon S3<br/>(Static Assets / Archive)"]
    BKP["Cross-AZ / Cross-Region<br/>Backups & Snapshots"]

    IAM["IAM<br/>(Least Privilege)"]
    CI["IaC / CI-CD"]

    U2 --> CDN --> WAF --> ALB
    ALB --> ASG --> APP2

    APP2 --> CACHE
    APP2 --> RDSW
    RDSW --> RDSR

    APP2 --> S3
    RDSW --> BKP

    IAM -.-> APP2
    CI -.-> ASG
  end

