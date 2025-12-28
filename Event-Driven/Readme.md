# Project 3: Scalable Event-Driven Ingestion & Processing Pipeline

## 1. Overview: The SaaS Evolution
This project demonstrates the transformation of a traditional "Single-Tenant" synchronous system into a modern, **Multi-Tenant SaaS architecture**. 

In the previous model (Project 1), each client had a dedicated stack with predictable traffic. This project tackles the challenges of **unpredictable bursts** and **distributed consistency** by migrating to an asynchronous, Event-Driven Architecture (EDA).

---

## 2. Architecture Diagram

```mermaid
flowchart TB
    %% =====================
    %% Ingestion Layer
    %% =====================
    subgraph Ingestion ["Ingestion Layer (SaaS API)"]
        direction TB
        UI["Web / Mobile Clients"] --> AGW["Amazon API Gateway"]
        AGW --> L_Entry["Lambda: Request Validator\n(Idempotent)"]
    end

    %% =====================
    %% Decoupling Layer
    %% =====================
    subgraph Decoupling ["Async Messaging (The Buffer)"]
        direction TB
        L_Entry --> SQS["Amazon SQS (Task Queue)\n(Scalable, Durable)"]
        SQS -.-> DLQ["Dead Letter Queue (Fault Tolerance)"]
    end

    %% =====================
    %% Processing Tier
    %% =====================
    subgraph Processing ["Processing Tier"]
        direction TB
        SQS --> L_Process["Lambda: Business Engine\n(Idempotent)"]
        L_Process --> DDB[("Amazon DynamoDB (Storage)\nServerless, Low-Latency)")]
    end

    %% =====================
    %% Observability Layer
    %% =====================
    subgraph Observability ["Monitoring & Alerting"]
        direction TB
        L_Entry --> CW_Lambda["CloudWatch Metrics/Logs"]
        L_Process --> CW_Biz["CloudWatch Metrics/Logs"]
        DLQ --> SNS_Alert["SNS Notification / PagerDuty"]
        CW_Lambda -->|Alerts| SNS_Alert
        CW_Biz -->|Alerts| SNS_Alert
    end

    %% =====================
    %% Styling
    %% =====================
    style SQS fill:#fff7e6,stroke:#ffa940,stroke-width:2px
    style DLQ fill:#fff1f0,stroke:#ffa39e
    style DDB fill:#e6f7ff,stroke:#1890ff
    style SNS_Alert fill:#ffd8bf,stroke:#fa8c16,stroke-width:2px
```
