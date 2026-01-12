# Project 1: LAMP Stack Modernization & Cloud-Native Evolution

## 🌟 Executive Summary
This project demonstrates the transition of a legacy **LAMP (Linux, Apache, MySQL, PHP)** monolith from an on-premises data center to a highly optimized, **2026-standard AWS Cloud-Native architecture**. 

---

## 🏗️ Architecture Evolution

### 1. Core Business Path (L1 View)
The primary traffic flow focuses on high availability, read/write splitting, and **Static Asset Offloading**.

```mermaid
flowchart TB
    U2["Global User (Dual-Stack)"]
    CF["CloudFront + Origin Shield"]
    ALB["ALB (L7 Load Balancer)"]
    ASG["ASG (m8g Graviton4 Compute)"]
    S3["S3 (Static Assets)"]
    Proxy["RDS Proxy"]
    
    subgraph Aurora ["Aurora Cluster (Multi-AZ)"]
        direction LR
        RW["Primary RW"] -.->|Sync| RO["Reader RO"]
    end

    U2 --> CF
    CF -- "Dynamic Content" --> ALB --> ASG
    CF -- "Static Content" --> S3
    ASG -- "Uploads/Assets" --> S3
    ASG --> Proxy
    Proxy --> RW
    Proxy -.-> RO

🛡️ Production-Grade Details (Deep Dive)<details><summary><b>🚀 Click to Expand: Full Expert View (Security, Observability & Governance)</b></summary>This comprehensive diagram illustrates the "hidden" infrastructure required for enterprise compliance, including envelope encryption, private connectivity, and distributed tracing.代码段flowchart TB
    subgraph O ["✅ 2026 OPTIMIZED BLUEPRINT"]
        direction TB
        U2["Global User (Dual-Stack)"]
        
        subgraph Traffic_Tier ["Edge & Compute"]
            CF["CloudFront + Origin Shield"]
            WAF["AWS WAF"]
            ALB["ALB (Dual-Stack)"]
            ASG["ASG (m8g Graviton4)"]
            U2 --> CF --> WAF --> ALB --> ASG
        end

        subgraph D_Tier ["Data & Storage"]
            direction LR
            S3["S3 Intelligent-Tiering"]
            Proxy["RDS Proxy"]
            subgraph Aurora ["Aurora Serverless v2"]
                direction LR
                RDSW["Primary"] -.->|Sync| RDSR["Reader"]
            end
            ASG --> Proxy --> Aurora
            ASG -- "Sync Media" --> S3
            CF -- "Cached Assets" --> S3
        end

        subgraph M_Tier ["Management & Observability"]
            direction TB
            VPCE["VPC Endpoints (PrivateLink)"]
            CWL["CloudWatch Logs/Metrics"]
            XRY["X-Ray Traces"]
            KMS["KMS (Envelope Encryption)"]
            SSM["SSM Session Manager"]
        end

        %% Connections
        ASG -.-> VPCE
        VPCE -.-> S3
        VPCE -.-> KMS
        ASG -.-> CWL
        ASG -.-> XRY
        S3 --> GLC["Glacier Deep Archive"]
    end
</details>💎 Technical Pillars & Optimization1. Advanced Networking (ANS-C01 Optimized)IPv6-First Strategy: To mitigate the costs associated with Public IPv4 addresses ($0.005/hr/IP), the backend operates on IPv6-Only subnets.Egress-Only IGW: Provides secure, stateful outbound-only communication for IPv6 instances without the hourly costs or bandwidth bottlenecks of a NAT Gateway.VPC Endpoints (PrivateLink): S3 and KMS traffic remains entirely within the AWS backbone, reducing latency and exposure to the public internet.2. Compute & Cost GovernanceGraviton4 (m8g) Adoption: Migrating from x86 to ARM64 (Graviton4) yielded a 40% improvement in price-performance for PHP-FPM workloads.Stateless Architecture: By offloading media and static assets to S3, EC2 instances in the ASG become completely stateless, enabling seamless scaling and replacement.3. Data Integrity & Disaster Recovery (SAP-C02 Standard)RDS Proxy: Implements connection pooling to handle PHP's ephemeral connection nature and reduces failover time by up to 66%.S3 Intelligent-Tiering: Automatically optimizes storage costs for static assets (wp-content) without manual lifecycle management intervention.Aurora Global Forwarding: Prepared for future multi-region expansion by allowing read replicas to forward write requests to the primary region.📉 Architecture Decision Records (ADR)<details><summary><b>📊 Click to View Detailed Technology Selections</b></summary>CategorySelectionJustificationComputeGraviton4 (m8g)Best-in-class performance for PHP 8.x JIT; 20% lower cost than R7i.DatabaseAurora Serverless v2Instant scaling (0.5 to 128 ACUs) to match LAMP bursty traffic patterns.SecurityWAF + ShieldEdge-layer mitigation of SQLi and XSS before traffic enters the VPC.OperationsSSM (No-SSH)Zero-open-port policy; all administrative access via IAM-authenticated sessions.StorageS3 + CloudFrontDecoupling static assets from the compute layer to enable stateless EC2.
