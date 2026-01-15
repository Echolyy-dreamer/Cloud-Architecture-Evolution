graph TD
    subgraph AWS_Region_us-east-1
        direction TB
        
        subgraph Public_Subnet_Tier
            direction LR
            ALB[Application Load Balancer]
        end

        subgraph AZ_A_us-east-1a
            direction TB
            subgraph Private_Subnet_App_A
                EC2_A[EC2: WordPress Instance A]
            end
            subgraph Private_Subnet_Data_A
                RDS_A[(Aurora Writer)]
                Redis_A[ElastiCache Redis]
            end
        end

        subgraph AZ_B_us-east-1b
            direction TB
            subgraph Private_Subnet_App_B
                EC2_B[EC2: WordPress Instance B]
            end
            subgraph Private_Subnet_Data_B
                RDS_B[(Aurora Reader/Replica)]
                Redis_B[ElastiCache Redis]
            end
        end

        %% 流量流向
        User((Internet User)) -->|HTTPS:443| ALB
        ALB -->|Target Group| EC2_A
        ALB -->|Target Group| EC2_B
        
        %% 数据库代理层
        EC2_A & EC2_B -->|SQL:3306| Proxy[RDS Proxy]
        Proxy -->|Connection Pool| RDS_A
        
        %% 缓存层
        EC2_A & EC2_B -->|Port:6379| Redis_A
        
        %% 外部服务
        EC2_A & EC2_B -.->|IAM Role| Secrets[Secrets Manager]
        EC2_A & EC2_B -.->|Media Offload| S3[S3 Bucket]
    end
