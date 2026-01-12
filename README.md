# Healthcare AI Model Validation Platform

A cloud-based architecture for secure external validation of AI models in healthcare settings, enabling collaboration between AI developers and hospitals without direct data sharing.

## Problem Statement

AI models for healthcare diagnosis face critical generalization challenges:

- **Narrow Training Data**: Datasets often fail to capture real-world patient population variability
- **Overfitting**: Even with "unseen" validation sets, data from single institutions creates hidden biases
- **Dataset Shifts**: Covariate shift (imaging protocols, devices) and concept shift (disease presentation across populations)
- **Real-world Failures**: Models with strong internal metrics fail in external environments (different hospitals, regions, demographics)

### Case Studies
- **Google Health (2018-2019)**: Diabetic retinopathy AI achieved >90% accuracy in controlled settings but struggled with real-world image quality in Thai clinics
- **Mass Eye & Ear / Harvard**: Glaucoma assessment AI trained on public data showed dramatic R and R² coefficient drops on hospital data

## The Solution

A secure cloud-based platform enabling:
1. **AI Companies**: Host models on secure cloud instances (AWS EC2) with evaluation APIs
2. **Hospitals**: Send patient data through APIs, receive outputs, assess performance locally
3. **Validation**: Report accuracy metrics back to developers without exposing sensitive datasets

## Design Principle

> **Zero-Cost-at-Idle**: When there are no clients and no evaluations requested, AWS charges = $0

All infrastructure is 100% serverless. See [Cost Model](docs/COST_MODEL.md) for details.

## Project Structure

```
healthcare-ai-validation/
├── docs/
│   ├── ARCHITECTURE.md          # System architecture design
│   ├── COLLABORATION_OPTIONS.md # Ways to work with research scientists
│   ├── BUILDABLE_COMPONENTS.md  # What we can build
│   ├── COST_MODEL.md            # Zero-cost-at-idle design
│   └── FOCUS_AREAS.md           # Priority areas for development
├── diagrams/
│   └── architecture.mmd         # Mermaid architecture diagram
├── src/                         # Source code (future)
└── README.md
```

## Quick Links

- [Cost Model (Zero-Cost-at-Idle)](docs/COST_MODEL.md)
- [Collaboration Options](docs/COLLABORATION_OPTIONS.md)
- [System Architecture](docs/ARCHITECTURE.md)
- [Buildable Components](docs/BUILDABLE_COMPONENTS.md)
- [Focus Areas](docs/FOCUS_AREAS.md)

## Key Technologies

- **AWS HealthLake**: FHIR-based healthcare data storage
- **AWS SageMaker**: ML model deployment and inference
- **AWS API Gateway**: Secure API endpoints
- **AWS KMS**: Encryption key management
- **HIPAA-eligible services**: Compliance framework

## Context

This project is prepared for collaboration with:

**Dr. Mohammad Eslami, Ph.D., SMIEEE**  
Mass Eye and Ear  
Instructor in Ophthalmology  
Harvard Medical School

## Development

Built by **Amir Olyaei**  
Technical Architect  
[DevArts Lab](https://notion.so/61c6b79808ce476290c753165851b070)

## License

© 2026 Amir Olyaei / DevArts Lab. All Rights Reserved.

This source code is provided for reference and educational purposes only. No permission is granted to use, copy, modify, merge, publish, distribute, sublicense, or sell copies of this software. See [LICENSE](LICENSE) for details.
