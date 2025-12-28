## Overview

This design demonstrates how a small startup can evolve from a single on-premises LAMP server into a resilient, scalable, and AWS-native architecture — without over-provisioning and while preserving operational simplicity.

Rather than assuming known traffic patterns or steady growth, this architecture is intentionally designed for **uncertain and non-linear scaling**, allowing the system to grow naturally as the business evolves.

### Architecture Diagram
```mermaid
graph TD
    %% =========================
    %% Traditional Architecture
    %% =========================
    subgraph Legacy ["❌ Traditional Monolith (On-Prem)"]
        direction TB
        U1["User (Global)"] --> LB1["Single LB / No LB"]
        LB1 --> APP1["Coupled Web/App (LAMP)"]
        APP1 --> DB1[("Local MySQL")]
        APP1 --> FS1["Local File System"]
    end

    %% =========================
    %% Optimized Architecture
    %% =========================
    subgraph Cloud ["✅ AWS Cloud-Native (Evolution)"]
        direction TB
        U2["User (Global)"] --> CDN["CloudFront (Edge)"]
        CDN --> WAF["WAF / Shield"]
        WAF --> ALB["ALB (Layer 7)"]
        
        subgraph Compute ["Scalable Tier"]
            ALB --> ASG["Auto Scaling Group"]
            ASG --> EC2["Stateless EC2/Containers"]
        end

        subgraph Storage ["Data Tier"]
            EC2 --> Redis[("ElastiCache (Session)")]
            EC2 --> RDS_W[("RDS Writer")]
            RDS_W -.-> RDS_R[("RDS Read Replica")]
            EC2 --> S3[("S3 Object Storage")]
        end
    end

    %% Style
    style Legacy fill:#fff1f0,stroke:#ffa39e
    style Cloud fill:#f6ffed,stroke:#b7eb8f
    style RDS_W fill:#e6f7ff
    style RDS_R fill:#e6f7ff

