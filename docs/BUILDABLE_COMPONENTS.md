# Buildable Components

## Overview

Based on the research and Dr. Eslami's context, here are concrete components we can build, ordered by complexity and value.

### Design Constraints
- **Zero-cost-at-idle**: All infrastructure must be serverless/pay-per-use
- **AI-assisted development**: Timelines reflect accelerated development with AI coding agents

---

## Tier 1: Quick Wins (1-2 weeks)

### 1.1 Model Evaluation SDK (Python)

**What**: A Python library that hospitals install to evaluate external AI models.

**Features**:
- Standardized API client for model inference
- Built-in data anonymization helpers
- DICOM image preprocessing
- Metrics calculation (AUC, sensitivity, specificity, etc.)
- Automated report generation

**Tech Stack**:
```
- Python 3.9+
- httpx (async HTTP client)
- pydicom (medical imaging)
- scikit-learn (metrics)
- pandas (data handling)
- Jinja2 (report templates)
```

**Example Usage**:
```python
from healthcare_eval import ModelEvaluator, DICOMLoader

# Load local hospital data
loader = DICOMLoader("/path/to/retinal/images")
data = loader.load_anonymized()

# Connect to external model
evaluator = ModelEvaluator(
    model_endpoint="https://api.example.com/v1/diabetic-retinopathy",
    api_key="hospital_key_xxx"
)

# Run evaluation
results = evaluator.evaluate(data, ground_truth=labels)

# Generate report
results.to_pdf("dr_model_validation_report.pdf")
```

**Deliverables**:
- PyPI package
- Documentation site
- Example notebooks

---

### 1.2 Metrics Dashboard (Web)

**What**: Web dashboard for tracking model performance across evaluations.

**Features**:
- Model performance over time
- Demographic subgroup analysis
- Bias detection visualizations
- Comparison across models
- Export for regulatory submissions

**Tech Stack**:
```
- Next.js 14 (React framework)
- Tailwind CSS + shadcn/ui
- Recharts (visualizations)
- DynamoDB On-Demand (backend) - $0 at idle
- S3 + CloudFront (static hosting) - minimal storage cost only
```

**Deliverables**:
- Deployed web application
- REST API for data ingestion
- PDF export functionality

---

## Tier 2: Core Platform (2-4 weeks)

### 2.1 Model Hosting Service (Serverless)

**What**: Secure infrastructure for AI developers to deploy models for evaluation.

**Features**:
- Docker container upload and registry
- **Serverless** inference endpoints (scale to zero)
- API key management per hospital
- Usage tracking and billing hooks
- Model versioning

**Tech Stack (Zero-Cost-at-Idle)**:
```
- SageMaker SERVERLESS Inference (NOT real-time endpoints)
- Amazon ECR (container registry) - storage only
- API Gateway + Lambda (API layer) - per-request
- Cognito (authentication) - per MAU
- DynamoDB On-Demand (metadata) - per-request
```

**⚠️ CRITICAL**: Do NOT use SageMaker Real-time Endpoints (always-on costs)

**Architecture**:
```
Developer                Platform                    Hospital
    │                        │                           │
    │  Upload model.tar.gz   │                           │
    │  ────────────────────▶ │                           │
    │                        │                           │
    │  Get endpoint URL      │                           │
    │  ◀──────────────────── │                           │
    │                        │   Request API key         │
    │                        │ ◀──────────────────────── │
    │                        │                           │
    │                        │   Invoke model            │
    │                        │ ◀──────────────────────── │
    │                        │                           │
    │                        │   Return predictions      │
    │                        │ ────────────────────────▶ │
```

---

### 2.2 Data Shift Detection Service

**What**: Automated monitoring for distribution shifts that may affect model performance.

**Features**:
- Input data distribution monitoring
- Statistical drift detection (KS test, MMD, etc.)
- Alerting when drift exceeds thresholds
- Recommendations for recalibration
- Historical drift tracking

**Tech Stack (Serverless)**:
```
- Python (drift detection algorithms)
- AWS Lambda (processing) - $0 at idle
- CloudWatch (alerting) - basic tier free
- S3 (baseline distributions) - storage only
- SNS (notifications) - per-message
```

**Use Case**:
```
Scenario: Hospital upgrades retinal imaging device
→ System detects image characteristics shift
→ Alert sent: "Model may need recalibration"
→ Recommends re-evaluation on new device data
```

---

### 2.3 Anonymization Pipeline

**What**: Automated de-identification of medical data before external API calls.

**Features**:
- DICOM header scrubbing
- Face/identifying feature detection and removal
- Text PHI redaction (for clinical notes)
- Audit trail of transformations
- HIPAA Safe Harbor compliance

**Tech Stack (Serverless)**:
```
- pydicom (DICOM manipulation)
- OpenCV / MediaPipe (face detection)
- Amazon Comprehend Medical (text PHI) - per-request
- AWS Lambda (processing) - $0 at idle
```

---

## Tier 3: Advanced Features (1-2 months)

### 3.1 Federated Learning Orchestrator

**What**: Platform for coordinating federated model training across multiple hospitals.

**Features**:
- Central aggregation server
- Secure client selection
- Differential privacy integration
- Progress monitoring dashboard
- Model convergence tracking

**Tech Stack (Cost-Conscious)**:
```
- NVIDIA FLARE or Flower (FL framework)
- AWS Lambda + Step Functions (orchestration) - NOT EKS
- gRPC via API Gateway
- TensorFlow/PyTorch (model training on-demand)
```

**Note**: FL training runs on-demand only, no always-on infrastructure.
**Complexity**: High - requires buy-in from multiple institutions.

---

### 3.2 FDA Submission Report Generator

**What**: Automated generation of documentation required for FDA 510(k) AI/ML submissions.

**Features**:
- Performance summary generation
- Bias analysis reports
- Training data characterization
- Intended use statement templates
- Change protocol documentation

**Regulatory Context**:
- FDA requires evidence of performance across intended populations
- External validation data strengthens submissions
- Standardized reporting reduces approval friction

---

### 3.3 AWS Marketplace Integration

**What**: Package the platform for distribution via AWS Marketplace.

**Features**:
- One-click deployment for hospitals
- Private offers for enterprise pricing
- Usage-based billing integration
- Managed updates and support

**Reference**: Dr. Eslami mentioned "AWS Model Private Offer seems very interesting"

---

## Priority Recommendation

Based on immediate value and feasibility (AI-assisted timelines):

| Priority | Component | Effort | Impact | Idle Cost |
|----------|-----------|--------|--------|----------|
| **1** | Model Evaluation SDK | 3-5 days | High | $0 (client-side) |
| **2** | Metrics Dashboard | 3-5 days | High | ~$0.02/GB |
| **3** | Anonymization Pipeline | 2-3 days | Medium | $0 |
| **4** | Model Hosting Service | 1-2 weeks | High | ~$0.50/month |
| **5** | Data Shift Detection | 3-5 days | Medium | $0 |
| **6** | Federated Learning | 2-4 weeks | High | $0 |

---

## MVP Definition

**Minimum Viable Product** for initial collaboration:

1. **Python SDK** for model evaluation
2. **Simple web dashboard** for viewing results
3. **One demo model** (e.g., diabetic retinopathy or glaucoma)
4. **Validation on MEEI data** (with Dr. Eslami)

This proves the concept before building full infrastructure.
