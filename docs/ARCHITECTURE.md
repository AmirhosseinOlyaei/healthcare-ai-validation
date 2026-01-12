# System Architecture

## High-Level Architecture

The platform connects three key stakeholders:
1. **AI Model Developers** - Companies building healthcare AI
2. **Healthcare Institutions** - Hospitals with real-world patient data
3. **Platform** - Secure cloud infrastructure mediating validation

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           HEALTHCARE AI VALIDATION PLATFORM                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────┐         ┌──────────────────┐         ┌──────────────┐ │
│  │   AI DEVELOPER   │         │     PLATFORM     │         │   HOSPITAL   │ │
│  │                  │         │                  │         │              │ │
│  │  ┌────────────┐  │         │  ┌────────────┐  │         │ ┌──────────┐ │ │
│  │  │ ML Model   │──┼────────▶│  │ Model      │  │         │ │ EHR/PACS │ │ │
│  │  │ Container  │  │ Deploy  │  │ Registry   │  │         │ │ Systems  │ │ │
│  │  └────────────┘  │         │  └─────┬──────┘  │         │ └────┬─────┘ │ │
│  │                  │         │        │         │         │      │       │ │
│  │  ┌────────────┐  │         │  ┌─────▼──────┐  │         │ ┌────▼─────┐ │ │
│  │  │ Model      │  │         │  │ SageMaker  │  │◀────────┼─│ Eval     │ │ │
│  │  │ Metadata   │──┼────────▶│  │ Endpoints  │  │  API    │ │ Client   │ │ │
│  │  └────────────┘  │         │  └─────┬──────┘  │         │ └────┬─────┘ │ │
│  │                  │         │        │         │         │      │       │ │
│  │  ┌────────────┐  │         │  ┌─────▼──────┐  │         │ ┌────▼─────┐ │ │
│  │  │ Dashboard  │◀─┼─────────│  │ Results    │  │────────▶│ │ Local    │ │ │
│  │  │ (Results)  │  │ Reports │  │ Aggregator │  │ Metrics │ │ Metrics  │ │ │
│  │  └────────────┘  │         │  └────────────┘  │         │ └──────────┘ │ │
│  │                  │         │                  │         │              │ │
│  └──────────────────┘         └──────────────────┘         └──────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Model Hosting Layer

```
┌─────────────────────────────────────────────────────────┐
│                   MODEL HOSTING LAYER                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │   Amazon    │  │   Amazon    │  │    AWS Lambda   │  │
│  │   ECR       │  │  SageMaker  │  │   (Serverless)  │  │
│  │  (Container │  │  (Managed   │  │                 │  │
│  │   Registry) │  │   Hosting)  │  │                 │  │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘  │
│         │                │                   │           │
│         └────────────────┼───────────────────┘           │
│                          ▼                               │
│               ┌──────────────────┐                       │
│               │   API Gateway    │                       │
│               │   (REST/GraphQL) │                       │
│               └──────────────────┘                       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Technologies:**
- **Amazon ECR**: Container registry for model images
- **Amazon SageMaker**: Managed ML inference endpoints
- **AWS Lambda**: Serverless inference for lightweight models
- **API Gateway**: Secure, rate-limited API access

### 2. Security & Compliance Layer

```
┌─────────────────────────────────────────────────────────┐
│               SECURITY & COMPLIANCE LAYER                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │    AWS      │  │    AWS      │  │    AWS          │  │
│  │    IAM      │  │    KMS      │  │    WAF          │  │
│  │  (Identity) │  │ (Encryption)│  │  (Firewall)     │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │    VPC      │  │  CloudTrail │  │   CloudWatch    │  │
│  │  (Network   │  │  (Audit     │  │  (Monitoring)   │  │
│  │  Isolation) │  │   Logging)  │  │                 │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              HIPAA COMPLIANCE FRAMEWORK           │   │
│  │  • BAA with AWS  • PHI Encryption  • Access Logs │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Key Security Features:**
- **HIPAA BAA**: Business Associate Agreement with AWS
- **Encryption**: TLS 1.2+ in transit, AES-256 at rest
- **Network Isolation**: VPC with private subnets
- **Audit Trails**: Complete logging via CloudTrail

### 3. Hospital Integration Layer

```
┌─────────────────────────────────────────────────────────┐
│              HOSPITAL INTEGRATION LAYER                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              EVALUATION CLIENT SDK               │    │
│  │  • Python  • JavaScript  • REST API             │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│                          ▼                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │   Data      │  │   DICOM     │  │    FHIR         │  │
│  │   Anonymizer│  │   Handler   │  │    Adapter      │  │
│  │             │  │             │  │                 │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              LOCAL METRICS CALCULATOR            │    │
│  │  • Confusion Matrix  • AUC/ROC  • Calibration   │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Components:**
- **SDK**: Multi-language client libraries
- **Data Anonymizer**: Remove PHI before API calls
- **DICOM Handler**: Medical imaging format support
- **FHIR Adapter**: Healthcare interoperability standard
- **Metrics Calculator**: Local computation of performance

### 4. Results & Reporting Layer

```
┌─────────────────────────────────────────────────────────┐
│              RESULTS & REPORTING LAYER                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │   DynamoDB  │  │     S3      │  │    Redshift     │  │
│  │  (Results   │  │  (Reports   │  │   (Analytics)   │  │
│  │   Store)    │  │   Storage)  │  │                 │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
│                          │                               │
│                          ▼                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │              REPORTING DASHBOARD                 │    │
│  │  • Model Performance  • Bias Analysis           │    │
│  │  • Demographic Breakdowns  • Trend Tracking     │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │           FDA SUBMISSION REPORT GENERATOR        │    │
│  │  • 510(k) Format  • Performance Summary         │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Data Flow

### Evaluation Flow (No PHI Leaves Hospital)

```
Hospital                    Platform                    Developer
   │                           │                            │
   │  1. Request model access  │                            │
   │  ─────────────────────▶   │                            │
   │                           │                            │
   │  2. API credentials       │                            │
   │  ◀─────────────────────   │                            │
   │                           │                            │
   │  3. Send anonymized data  │                            │
   │  ─────────────────────▶   │                            │
   │                           │  4. Inference request      │
   │                           │  (model hosted on platform)│
   │                           │                            │
   │  5. Predictions returned  │                            │
   │  ◀─────────────────────   │                            │
   │                           │                            │
   │  [LOCAL: Compare with     │                            │
   │   ground truth labels]    │                            │
   │                           │                            │
   │  6. Submit aggregate      │                            │
   │     metrics only          │                            │
   │  ─────────────────────▶   │                            │
   │                           │  7. Share performance      │
   │                           │     report                 │
   │                           │  ─────────────────────────▶│
   │                           │                            │
```

---

## AWS Services Summary

| Layer | Services | Purpose |
|-------|----------|---------|
| **Compute** | SageMaker, Lambda, EC2 | Model hosting & inference |
| **Storage** | S3, DynamoDB, Redshift | Data & results storage |
| **API** | API Gateway, AppSync | Secure API endpoints |
| **Security** | IAM, KMS, WAF, VPC | Access control & encryption |
| **Monitoring** | CloudWatch, CloudTrail | Logging & observability |
| **Healthcare** | HealthLake | FHIR data management |

---

## Deployment Options

### Option A: Fully Managed (SageMaker)
- Developers upload models to SageMaker
- Platform manages scaling, endpoints
- Best for: Standard ML models

### Option B: Container-Based (ECS/EKS)
- Developers provide Docker containers
- More flexibility for custom runtimes
- Best for: Complex or custom frameworks

### Option C: Serverless (Lambda)
- For lightweight models (<10GB)
- Pay-per-inference pricing
- Best for: Simple classification models

---

## Cost Considerations

Costs vary based on:
- Model complexity and inference time
- Number of API calls
- Data transfer volumes
- Storage requirements

*Refer to AWS pricing calculator for estimates. NDA prevents sharing specific figures.*
