# Cost Model: Zero-Cost-at-Idle Architecture

## Core Principle

> **When there are no clients and no evaluations requested, the AWS account incurs $0 in charges.**

This is the foundational design constraint for the entire platform.

---

## Architecture Decision: 100% Serverless

### ❌ Services We AVOID (Always-On Costs)

| Service | Why We Avoid | Monthly Cost if Idle |
|---------|--------------|---------------------|
| SageMaker Endpoints | Always-on instances | $50-500+/month |
| EC2 Instances | Always-on compute | $20-200+/month |
| RDS Databases | Always-on database | $15-100+/month |
| NAT Gateway | Per-hour charges | $32+/month |
| Elastic Load Balancer | Per-hour charges | $16+/month |
| Redshift | Always-on cluster | $180+/month |
| ElastiCache | Always-on cache | $12+/month |
| EKS Control Plane | Per-hour charges | $72/month |

### ✅ Services We USE (Pay-Per-Use Only)

| Service | Pricing Model | Cost at Idle |
|---------|---------------|--------------|
| **Lambda** | Per invocation + compute time | $0 |
| **API Gateway** | Per request | $0 |
| **DynamoDB On-Demand** | Per read/write request | $0 |
| **S3** | Per GB stored + requests | ~$0.02/GB/month (storage only) |
| **ECR** | Per GB stored | ~$0.10/GB/month (storage only) |
| **Cognito** | Per MAU (monthly active user) | $0 if no users |
| **CloudWatch** | Per metric/log ingested | $0 if no activity |
| **SageMaker Serverless** | Per inference request | $0 |
| **Step Functions** | Per state transition | $0 |

---

## Serverless Architecture Design

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ZERO-COST-AT-IDLE ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  REQUEST FLOW (only runs when evaluation requested):                    │
│                                                                         │
│  Hospital SDK                                                           │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────┐     ┌─────────────┐     ┌──────────────────────┐       │
│  │ API Gateway │────▶│   Lambda    │────▶│ SageMaker Serverless │       │
│  │ (per-request│     │ (per-invoke)│     │    Inference         │       │
│  │  pricing)   │     │             │     │  (per-inference)     │       │
│  └─────────────┘     └──────┬──────┘     └──────────────────────┘       │
│                             │                                           │
│                             ▼                                           │
│                      ┌─────────────┐                                    │
│                      │  DynamoDB   │                                    │
│                      │ (on-demand) │                                    │
│                      └─────────────┘                                    │
│                                                                         │
│  STORAGE (minimal idle cost):                                           │
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐                                    │
│  │     S3      │     │     ECR     │                                    │
│  │  (reports)  │     │  (models)   │                                    │
│  │ ~$0.02/GB   │     │ ~$0.10/GB   │                                    │
│  └─────────────┘     └─────────────┘                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Cost Breakdown by Usage

### Scenario: Complete Idle (No Usage)
| Component | Monthly Cost |
|-----------|--------------|
| Lambda | $0.00 |
| API Gateway | $0.00 |
| DynamoDB | $0.00 |
| Cognito | $0.00 |
| S3 (1GB reports) | $0.02 |
| ECR (5GB models) | $0.50 |
| **Total** | **~$0.52/month** |

### Scenario: Light Usage (100 evaluations/month)
| Component | Monthly Cost |
|-----------|--------------|
| Lambda (100 invocations, 10s each, 1GB memory) | $0.02 |
| API Gateway (100 requests) | $0.0004 |
| SageMaker Serverless (100 inferences) | $1-5 |
| DynamoDB (100 writes, 500 reads) | $0.001 |
| S3 + ECR storage | $0.52 |
| **Total** | **~$2-6/month** |

### Scenario: Active Usage (10,000 evaluations/month)
| Component | Monthly Cost |
|-----------|--------------|
| Lambda | $2 |
| API Gateway | $0.04 |
| SageMaker Serverless | $50-200 |
| DynamoDB | $0.10 |
| S3 + ECR storage | $1 |
| **Total** | **~$55-205/month** |

---

## Implementation Guidelines

### 1. Model Inference: SageMaker Serverless Inference

```python
# Deploy model with serverless config
from sagemaker.serverless import ServerlessInferenceConfig

serverless_config = ServerlessInferenceConfig(
    memory_size_in_mb=4096,  # 4GB
    max_concurrency=5,       # Max concurrent requests
)

model.deploy(
    serverless_inference_config=serverless_config,
    # NO instance_type - this is key!
)
```

**Pricing**: 
- $0.0001 per inference second
- No charges when not in use
- Cold start: 1-2 seconds (acceptable for batch evaluation)

### 2. API Layer: API Gateway + Lambda

```yaml
# serverless.yml
functions:
  evaluate:
    handler: handlers.evaluate
    memorySize: 1024
    timeout: 30
    events:
      - http:
          path: /evaluate
          method: POST
```

**Pricing**:
- API Gateway: $3.50 per million requests
- Lambda: $0.20 per million requests + $0.0000166667/GB-second

### 3. Database: DynamoDB On-Demand

```python
# Create table with on-demand billing
dynamodb.create_table(
    TableName='evaluation_results',
    BillingMode='PAY_PER_REQUEST',  # Critical: on-demand
    ...
)
```

**Pricing**:
- $1.25 per million write requests
- $0.25 per million read requests
- $0 when no requests

### 4. Dashboard: Static Site on S3 + CloudFront

```bash
# No server costs - static React/Next.js build
aws s3 sync ./build s3://dashboard-bucket/
```

**Pricing**:
- S3: $0.023/GB storage
- CloudFront: $0.085/GB transfer (only when accessed)

---

## Services to NEVER Use

| Service | Alternative |
|---------|-------------|
| SageMaker Real-time Endpoints | SageMaker Serverless Inference |
| EC2 | Lambda |
| RDS | DynamoDB On-Demand |
| ECS/EKS (always-on) | Lambda containers |
| NAT Gateway | Lambda in public subnet or VPC endpoints |
| ALB/NLB | API Gateway |
| ElastiCache | DynamoDB DAX (only if needed) |

---

## Cold Start Considerations

Serverless = cold starts. For this use case, it's acceptable:

| Component | Cold Start | Acceptable? |
|-----------|------------|-------------|
| Lambda | 100-500ms | ✅ Yes |
| SageMaker Serverless | 1-2 seconds | ✅ Yes (batch evaluation) |
| API Gateway | None | ✅ Yes |

**Why acceptable**: Hospital evaluations are batch processes, not real-time clinical decisions. A 2-second cold start is irrelevant when processing hundreds of images.

---

## Cost Monitoring & Alerts

Set up billing alerts to catch any issues:

```bash
# AWS CLI - create billing alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "MonthlyBudgetExceeded" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 86400 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT:billing-alerts
```

---

## Summary

| Principle | Implementation |
|-----------|----------------|
| **No always-on compute** | Lambda + SageMaker Serverless |
| **No always-on database** | DynamoDB On-Demand |
| **No always-on networking** | API Gateway (no ALB/NAT) |
| **Minimal storage costs** | S3 + ECR only |
| **Cost = f(usage)** | True pay-per-use |

**Result**: ~$0.50/month at idle, scaling linearly with actual usage.
