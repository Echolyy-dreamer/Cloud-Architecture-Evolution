
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

flowchart TB
    %% =========================
    %% Traditional Architecture
    %% =========================
    subgraph T["❌ 传统架构 (On-Prem / PC Based)"]
        direction TB
        U1["用户 (全球访问)"]
        LB1["无负载均衡 / 单点连接"]
        WEB1["单机 Web (Apache/PHP)"]
        DB1["单点数据库 (MySQL 5.7)"]
        FS1["本地磁盘存储 (No Backup)"]

        U1 --> LB1 --> WEB1 --> DB1
        WEB1 --> FS1
    end

    %% =========================
    %% Optimized Architecture
    %% =========================
    subgraph O["✅ 优化后的 AWS 云原生演进架构 (Blueprint)"]
        direction TB
        U2["用户 (全球)"]
        
        %% 接入与安全层
        CF["CloudFront CDN<br/>(IPv6 / Edge Cache)"]
        WAF["WAF + Shield<br/>(安全防护)"]
        ALB["Application Load Balancer<br/>(Dual-Stack / Multi-AZ)"]

        %% 计算层
        subgraph AppTier ["弹性计算层"]
            ASG["Auto Scaling Group"]
            APP2["Stateless PHP-FPM<br/>(EC2 t3.micro)"]
            SSM["SSM Session Manager<br/>(Secure Admin Access)"]
        end

        %% 数据访问与缓存层
        subgraph DataTier ["高性能数据层"]
            CACHE["ElastiCache Redis<br/>(Session & Object Cache)"]
            Proxy["RDS Proxy<br/>(Connection Pooling)"]
            
            subgraph AuroraCluster ["Aurora Serverless v2 (Encrypted)"]
                RDSW["Primary (Write)"]
                RDSR["Read Replica (Read)"]
            end
        end

        %% 存储层
        subgraph StorageTier ["持久化存储"]
            S3["Amazon S3<br/>(Static Assets)"]
            GLC["S3 Glacier<br/>(Archival > 6 Months)"]
            KMS["AWS KMS<br/>(CMK - Encrypted at Rest)"]
        end

        %% 逻辑连接
        U2 --> CF --> WAF --> ALB
        ALB --> ASG
        ASG --> APP2
        
        APP2 --> CACHE
        APP2 --> Proxy
        Proxy --> RDSW
        RDSW -.->|Asynchronous| RDSR
        
        APP2 --> S3
        S3 -->|Lifecycle Policy| GLC
        
        %% 安全与管理
        KMS -.->|Protects| AuroraCluster
        KMS -.->|Protects| S3
        SSM -.->|No-SSH Access| APP2
    end

    %% 演进箭头
    T ==>|Cloud Migration & Modernization| O

