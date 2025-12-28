flowchart TB

    %% =====================
    %% Global Users
    %% =====================
    subgraph Users ["🌍 Global Trading Clients"]
        direction TB
        U_LDN["London Clients"]
        U_SGP["Singapore Clients"]
    end

    %% =====================
    %% Global Entry
    %% =====================
    GA["AWS Global Accelerator<br/>(Anycast IP)"]

    U_LDN --> GA
    U_SGP --> GA

    %% =====================
    %% Primary Region
    %% =====================
    subgraph Region_LDN ["🇬🇧 Primary Region: London (eu-west-2)"]
        direction TB

        ALB_L["ALB"]
        ASG_L["Settlement Service<br/>(Active)"]
        DB_W[("Aurora Global DB<br/>Writer")]

        ALB_L --> ASG_L
        ASG_L --> DB_W
    end

    %% =====================
    %% Secondary Region
    %% =====================
    subgraph Region_SGP ["🇸🇬 Secondary Region: Singapore (ap-southeast-1)"]
        direction TB

        ALB_S["ALB"]
        ASG_S["Query Service<br/>(Read-Only)"]
        DB_R[("Aurora Global Replica")]

        ALB_S --> ASG_S
        ASG_S --> DB_R
    end

    %% =====================
    %% Traffic Routing
    %% =====================
    GA -->|Lowest Latency| ALB_L
    GA -->|Optimized Path| ALB_S

    %% =====================
    %% Global Sync & Control
    %% =====================
    DB_W ====>|Storage-level Replication < 1s| DB_R
    ASG_S -.->|Emergency Write / Failover| DB_W

    %% =====================
    %% Network Backbone
    %% =====================
    TGW_L["Transit Gateway<br/>London"]
    TGW_S["Transit Gateway<br/>Singapore"]

    TGW_L --- TGW_S

    %% =====================
    %% Styling
    %% =====================
    style DB_W fill:#e6f7ff,stroke:#1890ff,stroke-width:2px
    style DB_R fill:#fff7e6,stroke:#ffa940,stroke-width:2px
    style GA fill:#f9f0ff,stroke:#722ed1
