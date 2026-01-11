
## Overview

This design demonstrates how a small startup can evolve from a single on-premises server (e.g. workpress site) into a resilient, scalable, and AWS-native architecture — without over-provisioning and while preserving operational simplicity.

Rather than assuming known traffic patterns or steady growth, this architecture is intentionally designed for **uncertain and non-linear scaling**, allowing the system to grow naturally as the business evolves.

---

### Architecture Diagram

```mermaid
flowchart TB
    %% =========================
    %% Traditional Architecture
    %% =========================
    subgraph T["❌ 传统旧架构 (On-Prem)"]
        direction TB
        U1["用户 (全球访问)"]
        WEB1["单机 Web (Apache/PHP)<br/>所有请求混杂流向服务器"]
        DB1["单点数据库 (MySQL)"]
        FS1["本地磁盘<br/>(存储代码/媒体/Session)"]

        U1 --> WEB1
        WEB1 --> DB1
        WEB1 --> FS1
    end

    %% =========================
    %% Optimized Architecture
    %% =========================
    subgraph O["✅ 优化后的 AWS 蓝图架构"]
        direction TB
        U2["用户 (全球)"]
        
        %% 接入与安全层
        CF{"CloudFront<br/>(分流中心)"}
        WAF["WAF + Shield<br/>(边缘安全)"]
        
        %% 路径分流 - 静态
        subgraph StaticLayer ["存储层 (静态)"]
            S3["Amazon S3<br/>(媒体文件)"]
            GLC["S3 Glacier<br/>(归档规则)"]
        end

        %% 路径分流 - 动态
        subgraph ComputeLayer ["计算层 (动态)"]
            ALB["ALB 负载均衡"]
            ASG["Auto Scaling Group<br/>(无状态 EC2)"]
            SSM["SSM 管理端<br/>(无 SSH 秘钥)"]
        end

        %% 数据中心
        subgraph DataLayer ["数据中心 (高吞吐)"]
            CACHE["ElastiCache Redis<br/>(缓存/Session)"]
            Proxy["RDS Proxy<br/>(连接池)"]
            
            subgraph AuroraCluster ["Aurora Serverless v2"]
                RDSW["Primary (写)"]
                RDSR["Reader (读)"]
            end
        end

        %% 安全审计
        KMS["KMS 加密中心<br/>(CMK 托管)"]

        %% 核心逻辑连线 (动静分离细节)
        U2 --> CF
        CF --> WAF
        WAF -- "/wp-content/*" --> S3
        WAF -- "Default (*)" --> ALB
        
        %% 动态请求流转
        ALB --> ASG
        ASG --> CACHE
        ASG --> Proxy
        Proxy --> RDSW
        RDSW -.->|读写分离| RDSR
        
        %% 归档与加密流
        S3 -->|生命周期| GLC
        KMS -.->|静态加密| S3
        KMS -.->|静态加密| AuroraCluster
        SSM -.->|安全登录| ASG
    end

    %% 演进箭头
    T ==>|云原生现代化重构| O
