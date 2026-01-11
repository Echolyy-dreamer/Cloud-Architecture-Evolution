
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
        U1["Global User"]
        
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
        U2["Global User"]
        
        %% Edge Layer
        CF["CloudFront Content Delivery Network"]
        WAF["AWS WAF / Shield Security"]
        
        %% Storage Tier (Static Assets)
        subgraph S_Tier ["Storage & Archival Tier"]
            S3["Amazon S3 Bucket"]
            GLC["S3 Glacier Deep Archive"]
        end

        %% Compute Tier (Dynamic Application)
        subgraph C_Tier ["Elastic Compute Tier (Multi-AZ)"]
            ALB["Application Load Balancer"]
            ASG["Auto Scaling Group (EC2)"]
        end

        %% Data Tier (Persistence & Cache)
        subgraph D_Tier ["Distributed Data Tier"]
            CACHE["ElastiCache for Redis"]
            Proxy["RDS Proxy (Conn Pool)"]
            
            subgraph Aurora ["Aurora Serverless v2 Cluster"]
                RDSW["Primary Instance (RW)"]
                RDSR["Reader Instance (RO)"]
            end
        end

        %% Observability & Governance
        subgraph M_Tier ["Management & Observability"]
            CWL["CloudWatch Logs & Metrics"]
            XRY["AWS X-Ray Tracing"]
            KMS["KMS (Key Management)"]
            SSM["SSM Session Manager"]
            VPCE["VPC Endpoints (PrivateLink)"]
        end

        %% Core Traffic Routing (Path-Based)
        U2 --> CF
        CF --> WAF
        WAF -- "Static Path: /wp-content/*" --> S3
        WAF -- "Default Path: / (PHP)" --> ALB
        
        %% Internal Traffic Flow
        ALB --> ASG
        ASG --> CACHE
        ASG --> Proxy
        Proxy --> RDSW
        RDSW -.->|Multi-AZ Sync| RDSR
        
        %% Observability & Private Connectivity
        ASG -.->|Logs/Metrics| CWL
        ASG -.->|Traces| XRY
        ASG -.->|Private Access| VPCE
        VPCE -.-> S3
        VPCE -.-> KMS
        
        %% Security & Archival
        S3 -->|Lifecycle Policy| GLC
        KMS -.->|Envelope Encryption| S3
        KMS -.->|Envelope Encryption| Aurora
        SSM -.->|Secure Tunnel| ASG
    end

    %% Evolution Marker
    T ==>|Modernization & Cloud Transformation| O
