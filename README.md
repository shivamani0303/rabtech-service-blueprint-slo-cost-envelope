🚀 RabTech Task 02 — Service Blueprint, SLOs & Cloud Cost Envelope
> **Professional operational architecture and reliability design for a small two-tier web service.**
---
📌 Project Overview
This repository contains the operational design for a small cloud-based two-tier web service before infrastructure provisioning and deployment automation.
The project defines the service architecture, request and data flows, reliability objectives, failure recovery strategy, cloud cost envelope, security controls, and architecture decisions.
🎯 Key Deliverables
🏗️ Service architecture blueprint
🔄 Request and data flow documentation
🚀 Deployment flow
⚠️ Failure and recovery paths
📊 Service Level Indicators (SLIs)
🎯 Service Level Objectives (SLOs)
💥 Error budget and paging thresholds
💰 Three-level cloud cost model
🔐 Security and least-privilege design
♻️ Recovery Point Objective (RPO)
⏱️ Recovery Time Objective (RTO)
📝 Architecture Decision Records (ADRs)
---
🏗️ 1. Service Architecture
The proposed system follows a two-tier web service architecture.
Tier 1 — Web/API Application Tier
Responsible for:
Receiving HTTPS requests
Request validation
Business logic
Database operations
External API integration
Health checks
Logging and metrics
Graceful error handling
Tier 2 — Managed PostgreSQL Data Tier
Responsible for:
Persistent application data
Transaction processing
Automated backups
Database recovery
Data durability
Supporting Services
Component	Responsibility
DNS	Resolves the public service endpoint
Load Balancer	Routes traffic to healthy application instances
Web/API Tier	Processes business requests
PostgreSQL	Stores persistent application data
External APIs	Provides approved third-party services
Container Registry	Stores application images
CI/CD	Automates validation and deployment
Monitoring	Collects logs, metrics and alerts
Backup Storage	Stores protected database backups
---
🖼️ Architecture Blueprint
![Service Blueprint](architecture/service-blueprint.png)
Architecture Principles
🔒 Secure-by-default communication
📈 Horizontal application scalability
🗄️ Managed database service
♻️ Automated backups
🚨 Health checks and monitoring
🔑 Least-privilege access
🔐 Secrets stored outside source code
🌍 Recovery strategy based on business requirements
---
🔄 2. Request Flow
The normal request path is:
```text
User
  │
  │ HTTPS
  ▼
DNS / Internet
  │
  ▼
Load Balancer
  │
  ▼
Web / API Tier
  │
  ├──────────────► PostgreSQL
  │
  └──────────────► External API
  │
  ▼
Response
  │
  ▼
User
```
Request Sequence
User sends an HTTPS request.
DNS resolves the public service endpoint.
The load balancer receives the request.
The load balancer routes traffic to a healthy application instance.
The Web/API tier processes the request.
The application reads or writes PostgreSQL data when required.
The application calls an external dependency when required.
The response is returned to the user.
---
🔀 3. Data Flow
The primary data flow is:
```text
Web / API Tier
      │
      │ Read / Write
      ▼
PostgreSQL Database
      │
      │ Automated Backup
      ▼
Protected Backup Storage
```
External data flow:
```text
Web / API Tier
      │
      │ HTTPS API Request
      ▼
External Dependency
      │
      ▼
API Response
```
Data Protection
Database access is restricted to the application tier.
Database credentials are stored in a managed secrets service.
Backups are stored separately from application instances.
Sensitive values are never committed to source control.
Client traffic uses HTTPS/TLS.
---
🚀 4. Deployment Flow
The deployment pipeline follows:
```text
Developer
    │
    ▼
Source Repository
    │
    ▼
CI/CD Pipeline
    │
    ├──► Validation
    ├──► Tests
    └──► Security Checks
    │
    ▼
Container Build
    │
    ▼
Container Registry
    │
    ▼
Managed Orchestration
(ECS / Kubernetes / Equivalent)
    │
    ▼
Health Checks
    │
    ▼
Traffic Shift
    │
    ▼
New Version Live
```
Deployment Controls
Automated validation before deployment
Container image versioning
Health checks before traffic shifting
Rolling deployment strategy
Ability to roll back failed releases
No production secrets stored in source control
---
⚠️ 5. Failure & Recovery
The system is designed to detect and recover from common service failures.
Failure	Detection	Recovery
Database unavailable	Health checks and connection errors	Retry with exponential backoff and graceful failure
External API timeout	Timeout and error metrics	Timeout, retry where safe, circuit breaker and fallback
Application instance failure	Load balancer health check	Remove unhealthy instance and replace automatically
Deployment failure	CI/CD health checks	Roll back to previous known-good version
Regional outage	Monitoring and service checks	Failover according to selected recovery scenario
Backup failure	Backup monitoring	Alert and retry backup process
Recovery Philosophy
```text
Detect
  ↓
Contain
  ↓
Recover
  ↓
Validate
  ↓
Restore Normal Traffic
  ↓
Review & Improve
```
Detailed recovery procedures are documented in:
📄 `docs/failure-recovery.md`
---
📊 6. Service Level Indicators
The system measures the following key indicators.
SLI	Measurement
Availability	Successful requests / total valid requests
Latency	API response time, primarily P95
Error Rate	Failed requests / total requests
Database Availability	Successful database availability checks
Backup Success	Successful backups / scheduled backups
These indicators provide the measurement foundation for the SLO targets.
---
🎯 7. Service Level Objectives
The service uses scenario-specific reliability targets.
Metric	Lean	Balanced	Resilient
Availability	99.0%	99.5%	99.9%
P95 API Latency	<800 ms	<500 ms	<300 ms
Error Rate	<1%	<1%	<1%
Database Availability	≥99.9%	≥99.9%	≥99.9%
Backup Success	≥99%	≥99%	≥99%
---
💥 8. Error Budget
Error budget represents the amount of downtime permitted by the availability objective.
For a 30-day month:
Availability	Approx. Monthly Downtime
99.0%	7 hours 12 minutes
99.5%	3 hours 36 minutes
99.9%	43 minutes 12 seconds
Operational Principle
When the service consumes too much of its error budget:
🚨 Investigate reliability degradation
🛑 Reduce risky production changes
🔍 Prioritize reliability work
📈 Increase monitoring
🧪 Validate recovery procedures
---
🚨 9. Alerting & Paging
Alerts should be based on user impact and sustained degradation rather than individual transient failures.
Recommended Paging Conditions
Condition	Severity
Availability below SLO	🔴 Critical
Sustained high error rate	🔴 Critical
Severe P95 latency degradation	🟠 High
Database unavailable	🔴 Critical
Backup failure	🟠 High
External dependency degradation	🟡 Warning
Increased resource utilization	🟡 Warning
Alert Strategy
```text
Metric
  ↓
Threshold Evaluation
  ↓
Alert
  ↓
Severity Classification
  ↓
Notification / Pager
  ↓
Incident Response
```
---
💰 10. Cloud Cost Envelope
The project defines three infrastructure scenarios.
Scenario	Monthly Budget	Availability	P95 Latency	RPO	RTO
🟢 Lean	INR 2,500	99.0%	800 ms	24 hours	4 hours
🟡 Balanced	INR 8,000	99.5%	500 ms	60 minutes	60 minutes
🔴 Resilient	INR 25,000	99.9%	300 ms	15 minutes	30 minutes
Cost Allocation
The cost model considers:
Compute
Managed database
Storage
Network transfer
Logging and monitoring
Backup storage
Support
Contingency
Detailed calculations are available in:
📊 `cost-model/cloud-cost-model.xlsx`
> **Note:** Cost values are planning estimates based on the supplied scenario envelopes and are not provider invoices. Exact AWS, Azure or GCP pricing should be validated using the provider's current pricing calculator before production deployment.
---
♻️ 11. Recovery Objectives
Recovery requirements increase with the service reliability target.
Scenario	RPO	RTO
Lean	24 hours	4 hours
Balanced	60 minutes	60 minutes
Resilient	15 minutes	30 minutes
RPO — Recovery Point Objective
Maximum acceptable amount of data loss measured in time.
RTO — Recovery Time Objective
Maximum acceptable time required to restore the service.
---
🔐 12. Security Architecture
The design follows a defense-in-depth approach.
Transport Security
HTTPS/TLS for client communication
Encrypted service-to-service communication where applicable
Secrets
Secrets are not stored in source code
Managed secrets/configuration service
Credentials rotated according to operational requirements
Access Control
Least-privilege IAM/service accounts
Application tier only receives required database permissions
Administrative access is restricted
Production resources are separated from development access
Database Security
```text
Internet
   │
   X
   │
Load Balancer
   │
   ▼
Web / API Tier
   │
   │ Private Access
   ▼
PostgreSQL
```
The database should not be directly exposed to the public internet.
---
📈 13. Scalability
The application tier is designed to scale horizontally.
```text
                 Load Balancer
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       App #1      App #2      App #3
          │           │           │
          └───────────┼───────────┘
                      ▼
                  PostgreSQL
```
Scaling Strategy
Add application instances as traffic increases.
Use load balancing to distribute requests.
Monitor CPU, memory and request latency.
Scale based on sustained demand.
Evaluate database scaling independently.
---
🌍 14. Availability Strategy
The three scenarios provide progressively stronger resilience.
🟢 Lean
Optimized for low cost.
Minimal redundancy
Standard backups
Longer recovery objectives
Lower operational complexity
🟡 Balanced
Balanced between reliability and cost.
Increased application redundancy
More frequent backups
Improved monitoring
Faster recovery
🔴 Resilient
Optimized for high availability.
Multi-instance application deployment
Stronger redundancy
Frequent backups
Faster recovery
Enhanced monitoring
Regional recovery capability
---
📝 15. Architecture Decision Records
Architecture decisions are documented separately.
ADR	Decision
ADR-001	Cloud region selection
ADR-002	Managed PostgreSQL database
ADR-003	Secrets management
ADR-004	Least-privilege access
ADR-005	Recovery objectives
Location:
```text
adr/
├── ADR-001-cloud-region.md
├── ADR-002-database.md
├── ADR-003-secrets-management.md
├── ADR-004-least-privilege.md
└── ADR-005-recovery-objectives.md
```
---
📚 16. Documentation
Additional operational documentation:
Document	Purpose
`architecture/service-blueprint.md`	Detailed service architecture
`docs/failure-recovery.md`	Failure detection and recovery
`docs/assumptions.md`	Planning assumptions
`slo/slo-workbook.xlsx`	SLO and error-budget model
`cost-model/cloud-cost-model.xlsx`	Three-scenario cost model
---
📁 17. Repository Structure
```text
rabtech-service-blueprint-slo-cost-envelope/
│
├── README.md
│
├── architecture/
│   ├── service-blueprint.png
│   └── service-blueprint.md
│
├── slo/
│   └── slo-workbook.xlsx
│
├── cost-model/
│   └── cloud-cost-model.xlsx
│
├── adr/
│   ├── ADR-001-cloud-region.md
│   ├── ADR-002-database.md
│   ├── ADR-003-secrets-management.md
│   ├── ADR-004-least-privilege.md
│   └── ADR-005-recovery-objectives.md
│
├── docs/
│   ├── assumptions.md
│   └── failure-recovery.md
│
└── .gitignore
```
---
✅ 18. Task Completion Checklist
Requirement	Status
Service blueprint	✅ Complete
Request flow	✅ Complete
Data flow	✅ Complete
Deployment flow	✅ Complete
Failure paths	✅ Complete
Recovery strategy	✅ Complete
SLI definitions	✅ Complete
SLO targets	✅ Complete
Error budget	✅ Complete
Paging thresholds	✅ Complete
Three cost scenarios	✅ Complete
RPO / RTO	✅ Complete
Security controls	✅ Complete
Least privilege	✅ Complete
Secrets management	✅ Complete
Architecture Decision Records	✅ Complete
Assumptions	✅ Complete
---
🎓 19. Final Outcome
This project establishes the operational contract for the service before infrastructure and deployment automation are implemented.
The design connects:
```text
Business Requirements
        │
        ▼
Architecture
        │
        ▼
SLIs / SLOs
        │
        ▼
Error Budgets
        │
        ▼
Monitoring & Alerts
        │
        ▼
Recovery Objectives
        │
        ▼
Cloud Cost Envelope
        │
        ▼
Infrastructure Decisions
```
The selected infrastructure should be validated against the defined availability, latency, recovery and cost requirements rather than chosen independently.
---
👨‍💻 Project
RabTech Academy — Cloud Computing & DevOps
Task: 02 — Service Blueprint, SLOs & Cloud Cost Envelope
Repository: `rabtech-service-blueprint-slo-cost-envelope`
---
⭐ Designed with reliability, security, scalability and operational cost in mind.
