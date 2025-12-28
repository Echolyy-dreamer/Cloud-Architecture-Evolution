## Overview

This design demonstrates how a small startup can evolve from a single on-premises LAMP server into a resilient, scalable, and AWS-native architecture — without over-provisioning and while preserving operational simplicity.

Rather than assuming known traffic patterns or steady growth, this architecture is intentionally designed for **uncertain and non-linear scaling**, allowing the system to grow naturally as the business evolves.

### Architecture Diagram
```mermaid
flowchart LR
  %% =========================
  %% Traditional Architecture
  %% =========================
  subgraph T["❌ 传统三层 Web 架构"]
    U1["用户<br/>(全球访问)"]
    LB1["单一负载均衡<br/>(或无 LB)"]
    WEB1["Web 层<br/>(Apache/PHP)"]
    APP1["应用逻辑<br/>(紧耦合)"]
    DB1["单点数据库<br/>(MySQL)"]
    FS1["本地文件存储"]

    U1 --> LB1 --> WEB1 --> APP1 --> DB1
    APP1 --> FS1
  end

  %% =========================
  %% Optimized Architecture
  %% =========================
  subgraph O["✅ 优化后的云原生架构"]
    U2["用户<br/>(全球)"]

    CDN["CDN<br/>(静态资源加速)"]
    WAF["WAF + Shield"]
    ALB["ALB / NLB"]

    ASG["Auto Scaling<br/>Web/App 层"]
    APP2["Stateless App<br/>(容器 / EC2)"]

    CACHE["缓存层<br/>(Redis / ElastiCache)"]

    RDSW["RDS Writer"]
    RDSR["RDS Read Replica"]

    S3["对象存储 S3<br/>(静态/归档)"]
    BKP["跨区备份<br/>+ Snapshot"]

    IAM["IAM / Least Privilege"]
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

