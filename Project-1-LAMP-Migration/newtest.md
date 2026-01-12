# Project 1: LAMP Stack Modernization & Cloud-Native Evolution

## 🌟 Executive Summary

This project demonstrates how a legacy **LAMP (Linux, Apache, MySQL, PHP)** monolith evolves from an on-premises server to a production-grade AWS cloud-native architecture (2026-ready).

Focus areas:

* **IPv6-first cost governance**
* **Zero-trust & Private connectivity**
* **Stateless compute** and managed serverless data tiers

---

## 🏗️ Architecture Evolution

### 1️⃣ Overview

This diagram shows the evolution from a single on-prem instance to a scalable, observable AWS-native architecture.

```mermaid
flowchart TB
    %% ========================================================
    %% TRADITIONAL ARCHITECTURE (LEGACY)
    %% ========================================================
    subgraph T ["❌ TRADITIONAL ARCHITECTURE — LEGACY MONOLITH"]
        direction TB
        U1["Global User (IPv4 Only)"]
        subgraph OldServer ["Local Data Center / Single Instance"]
            WEB1["Web Tier: Apache & PHP"]
            DB1["Local MySQL Database"]
            FS1["Local Direct Attached Storage"]
        end
        U1 --> WEB1
        WEB1 --- DB1
        WEB1 --- FS1
    end

    %% ========================================================
    %% OPTIMIZED AWS BLUEPRINT
    %% ========================================================
    subgraph O ["✅ OPTIMIZED AWS CLOUD-NATIVE BLUEPRINT"]
        direction TB
        U2["Global User (Dual-Stack IPv4/v6)"]

        %% Edge Layer
        CF["CloudFront + Origin Shield"]
        WAF["AWS WAF (Edge Inspection)"]

        %% Storage Tier
        subgraph S_Tier ["Storage Tier"]
            S3["S3 Intelligent-Tiering"]
            GLC["S3 Glacier Deep Archive"]
        end

        %% Compute Tier
        subgraph C_Tier ["Compute Tier"]
            ALB["ALB (Dual-Stack)"]
            ASG["ASG (m8g Graviton4 + Warm Pool)"]
        end

        %% Distributed Data Tier
        subgraph D_Tier ["Distributed Data Tier"]
            direction TB
            CACHE["ElastiCache Redis (Serverless)"]
            Proxy["RDS Proxy"]
            subgraph Aurora ["Aurora Cluster (Multi-AZ)"]
                direction LR
                RDSW["Primary Node (RW)"]
                RDSR["Reader Node (RO)"]
                RDSW -. Sync_Replication .-> RDSR
            end
        end

        %% Management & Governance
        subgraph M_Tier ["Management & Governance"]
            VPCE["VPC Endpoints (PrivateLink)"]
            KMS["KMS (Envelope Encryption)"]
            SSM["SSM Session Manager"]
        end

        %% Observability Plane
        subgraph OBS ["Observability Plane (Request-Path Oriented)"]
            METRICS["CloudWatch Metrics"]
            LOGS["CloudWatch Logs"]
            TRACE["X-Ray Traces"]
        end

        %% Core Routing
        U2 --> CF
        CF --> WAF
        WAF -- Static_Path --> S3
        WAF -- Default_PHP_Path --> ALB

        %% Application Logic Flow
        ALB --> ASG
        ASG --> CACHE
        ASG --> Proxy
        Proxy -- Writes --> RDSW
        Proxy -. Reads_Scaling .-> RDSR

        %% Connectivity & Lifecycle
        ASG -.-> VPCE
        VPCE -.-> S3
        VPCE -.-> KMS
        S3 --> GLC

        %% Observability Bindings
        CF -. Edge_Trace .-> TRACE
        ALB -. Request_Trace .-> TRACE
        ASG -. App_Segments .-> TRACE
        Proxy -. DB_Segments .-> TRACE

        ASG -. App_Logs .-> LOGS
        ALB -. Access_Logs .-> LOGS
        WAF -. Security_Logs .-> LOGS

        CACHE -. Hit_Rate .-> METRICS
        Proxy -. Conn_Pool_Usage .-> METRICS
        RDSR -. Replica_Lag .-> METRICS
        ASG -. Scaling_Events .-> METRICS
    end
```

---

## 💎 Technical Pillars

1. **Advanced Networking**

   * IPv6-first subnets: Avoid NAT Gateway costs, deterministic latency.
   * PrivateLink (VPC Endpoints): Traffic to S3/KMS never leaves AWS backbone.

2. **Compute & Cost Governance**

   * Graviton4 (m8g): ~40% better price-performance for PHP-FPM.
   * Stateless Tiering: Offload mutable state to S3/Aurora, scale aggressively.

3. **Data Resilience**

   * RDS Proxy: Absorb PHP connection storms, seamless failover.
   * Aurora Serverless v2: Elastic scaling 0.5–128 ACUs, High Availability.
   * S3 Intelligent-Tiering: Lifecycle moves aged media to Glacier.

---

## 📊 Architecture Decision Records (ADR)

| Area     | Decision             | Reasoning                          |
| -------- | -------------------- | ---------------------------------- |
| Compute  | EC2 Graviton4 ASG    | Optimized for PHP JIT; lower TCO   |
| Database | Aurora Serverless v2 | Burst traffic; sub-minute failover |
| Security | WAF + PrivateLink    | Edge-layer filtering + Zero-Trust  |
| Ops      | SSM (No SSH)         | No Bastion, IAM-tracked access     |
| Storage  | S3 + CloudFront      | Stateless compute tier             |

---

## 🚀 Future Roadmap

* [ ] Infrastructure as Code: Terraform/CDK.
* [ ] Observability: Deep PHP tracing via X-Ray.
* [ ] Containerization: Migrate to EKS/Fargate.
* [ ] Global Scale: Aurora Global Database for sub-second cross-region DR.

