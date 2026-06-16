```mermaid
graph TD
    A[GitHub Actions Cron <br> 07:30 & 15:00 BST] -->|OIDC Auth Role| B(AWS CloudWatch API)
    A -->|OIDC Auth Role| ECS(AWS ECS API)
    
    B --> C{Resource Metrics}
    C -->|BytesInPerSec| D[MSK / Kafka Topics]
    C -->|Invocations/Errors| E[Lambda Functions]
    C -->|Delivery/Failures| F[SNS Topics]
    C -->|4xx/5xx/Count| H[API Gateway]
    
    ECS -->|Desired vs Running| G[ECS Clusters <br> Kafka Connect]
    
    D & E & F & G & H --> I[health_check.py <br> Orchestrator]
    I -->|Raw Data Objects| J[report_formatter.py]
    J -->|Markdown| K[GitHub Step Summary UI]
```
<img width="774" height="611" alt="image" src="https://github.com/user-attachments/assets/f5bb1ef6-376a-4ff0-8543-e1738038fe6c" />
<img width="1366" height="621" alt="image" src="https://github.com/user-attachments/assets/5cd2bec5-671f-4bd3-afa6-a346ea8cf270" />
<img width="1600" height="1190" alt="UC-3f433324-4ec9-4d7c-8e7a-dc2cbb4372ef" src="https://github.com/user-attachments/assets/c911d0b9-1fd5-44b3-ac5a-8d755e25b555" />


