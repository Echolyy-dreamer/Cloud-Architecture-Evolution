
## Overview

This design demonstrates how a small startup can evolve from a single on-premises LAMP server into a resilient, scalable, and AWS-native architecture — without over-provisioning and while preserving operational simplicity.

Rather than assuming known traffic patterns or steady growth, this architecture is intentionally designed for **uncertain and non-linear scaling**, allowing the system to grow naturally as the business evolves.

---

### Architecture Diagram

```mermaid
flowchart TB
  %% =========================
  %% Traditional Architecture
  %% =========================
  subgraph T["❌ 传统三层 Web 架构（单点 & 紧耦合）"]
    U1["用户<br/>(全球访问)"]
    LB1["单一负载均衡<br/>(或无 LB)"]
    WEB1["Web 层<br/>(Apache / PHP)"]
    APP1["应用逻辑<br/>(紧耦合)"]
    DB1["单点数据库<br/>(MySQL)"]
    FS1["本地文件存储"]

    U1 --> LB1 --> WEB1 --> APP1 --> DB1
    APP1 --> FS1
  end

  %% =========================
  %% Spacer
  %% =========================
  T --> O

  %% =========================
  %% Optimized Architecture
  %% =========================
  subgraph O["✅ 优化后的 AWS 云原生演进架构"]
    U2["用户<br/>(全球)"]

    CDN["CloudFront CDN<br/>(静态资源加速)"]
    WAF["WAF + Shield"]
    ALB["ALB / NLB"]

    ASG["Auto Scaling Group<br/>(Web / App)"]
    APP2["Stateless App<br/>(EC2 / Container)"]

    CACHE["缓存层<br/>(ElastiCache Redis)"]

    RDSW["RDS Writer"]
    RDSR["RDS Read Replica"]

    S3["Amazon S3<br/>(静态资源 / 归档)"]
    BKP["跨 Region 备份<br/>Snapshots / Backup"]

    IAM["IAM<br/>Least Privilege"]
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

