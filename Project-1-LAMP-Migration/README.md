
## Overview

This design demonstrates how a small startup can evolve from a single on-premises server (e.g. workpress site) into a resilient, scalable, and AWS-native architecture — without over-provisioning and while preserving operational simplicity.

Rather than assuming known traffic patterns or steady growth, this architecture is intentionally designed for **uncertain and non-linear scaling**, allowing the system to grow naturally as the business evolves.

---

### Architecture Diagram

```mermaid

flowchart TB
    %% ========================================================
    %% TRADITIONAL ARCHITECTURE (LEGACY)
    %% ========================================================
    subgraph T ["❌ TRADITIONAL ARCHITECTURE — LEGACY MONOLITH"]
        direction TB
        U1["Global User (IPv4 Only)"]
        
        subgraph OldServer ["Local Data Center / Single Instance ........................"]
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
        
        %% ======================
        %% Edge Layer
        %% ======================
        CF["CloudFront + Origin Shield"]
        WAF["AWS WAF (Edge Inspection)"]

        %% ======================
        %% Storage Tier
        %% ======================
        subgraph S_Tier ["Storage Tier"]
            S3["S3 Intelligent-Tiering"]
            GLC["S3 Glacier Deep Archive"]
        end

        %% ======================
        %% Compute Tier
        %% ======================
        subgraph C_Tier ["Compute Tier"]
            ALB["ALB (Dual-Stack)"]
            ASG["ASG (m8g Graviton4 + Warm Pool)"]
        end

        %% ======================
        %% Distributed Data Tier
        %% ======================
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

        %% ======================
        %% Management & Governance
        %% ======================
        subgraph M_Tier ["Management & Governance"]
            VPCE["VPC Endpoints (PrivateLink)"]
            KMS["KMS (Envelope Encryption)"]
            SSM["SSM Session Manager"]
        end

        %% ======================
        %% Observability Plane (关键补全)
        %% ======================
        subgraph OBS ["Observability Plane (Request-Path Oriented)"]
            METRICS["CloudWatch Metrics"]
            LOGS["CloudWatch Logs"]
            TRACE["X-Ray Traces"]
        end

        %% ======================
        %% Core Routing
        %% ======================
        U2 --> CF
        CF --> WAF
        WAF -- Static_Path --> S3
        WAF -- Default_PHP_Path --> ALB

        %% ======================
        %% Application Logic Flow
        %% ======================
        ALB --> ASG
        ASG --> CACHE

        ASG --> Proxy
        Proxy -- Writes --> RDSW
        Proxy -. Reads_Scaling .-> RDSR

        %% ======================
        %% Connectivity & Lifecycle
        %% ======================
        ASG -.-> VPCE
        VPCE -.-> S3
        VPCE -.-> KMS
        S3 --> GLC

        %% ======================
        %% Observability Bindings (信号即设计)
        %% ======================
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

    %% ========================================================
    %% Modernization Path
    %% ========================================================
    T ==>| Full-Stack Modernization | O
