```mermaid
graph LR
    User((终端用户)) -- HTTPS/443 --> CF{CloudFront 边缘节点}

    subgraph "CloudFront 路由逻辑 (Behaviors)"
        CF -- "Default (*) 转发到 ALB" --> ALB[ALB 负载均衡器]
        CF -- "/wp-content/uploads/* 转发到 S3" --> S3_Bucket[(S3 存储桶)]
    end

    subgraph "VPC 后端架构"
        ALB -- HTTP/80 --> EC2[WordPress EC2 实例]
        EC2 -- 数据库查询 --> RDS[(RDS MySQL)]
        EC2 -- 搬运图片 --> S3_Bucket
    end

    subgraph "数据流向说明"
        direction TB
        Upload[1. 用户上传图片] --> EC2
        Offload[2. 插件将图片同步至 S3] --> S3_Bucket
        Serve[3. 后续访问图片由 CF 从 S3 获取] --> User
    end
    
