# RabTech Task 02 — Service Blueprint, SLOs & Cloud Cost Envelope

## Project Overview

This repository contains the operational design for a small two-tier web service before infrastructure provisioning and deployment automation.

The project covers:

- Service architecture and request flow
- Application and database data flow
- Deployment flow
- Failure and recovery paths
- Service Level Indicators (SLIs)
- Service Level Objectives (SLOs)
- Error budgets and paging thresholds
- Three cloud cost scenarios
- Security and least-privilege controls
- Recovery Point Objectives (RPO)
- Recovery Time Objectives (RTO)
- Architecture Decision Records (ADRs)

---

## 1. Architecture

The service uses a two-tier architecture.

### Tier 1 — Web / API Application

Responsible for HTTPS requests, request validation, business logic, database operations, external API integration, health checks, logs, metrics and graceful error handling.

### Tier 2 — Managed PostgreSQL Database

Responsible for persistent application data, transactions, automated backups, data durability and recovery.

### Supporting Services

| Component | Responsibility |
| --- | --- |
| DNS | Resolves the public service endpoint |
| Load Balancer | Routes traffic to healthy application instances |
| Web / API Tier | Processes business requests |
| PostgreSQL | Stores persistent application data |
| External APIs | Provides approved third-party services |
| Container Registry | Stores container images |
| CI/CD | Validates, builds and deploys releases |
| Monitoring / Logging | Collects metrics, logs and alerts |
| Backup Storage | Stores protected database backups |

### Architecture Blueprint

![Service Blueprint](architecture/service-blueprint.png)

Detailed architecture: `architecture/service-blueprint.md`

---

## 2. Request Flow

```text
User
  |
  | HTTPS Request
  v
DNS / Internet
  |
  v
Load Balancer
  |
  v
Web / API Tier
  |
  +---------------> PostgreSQL Database
  |
  +---------------> External API
  |
  v
Response
  |
  v
User
```

### Request Sequence

1. User sends an HTTPS request.
2. DNS resolves the service endpoint.
3. Load balancer receives the request.
4. Load balancer routes it to a healthy application instance.
5. Application processes the request.
6. Application reads or writes PostgreSQL when required.
7. Application calls an external dependency when required.
8. Response is returned to the user.

---

## 3. Data Flow

```text
Web / API Tier
      |
      | Read / Write
      v
PostgreSQL Database
      |
      | Automated Backup
      v
Protected Backup Storage
```

External dependency:

```text
Web / API Tier
      |
      | HTTPS API Request
      v
External Dependency
      |
      v
API Response
```

### Data Protection

- Database access is restricted to the application tier.
- Secrets are stored outside source code.
- Backups are stored separately from application instances.
- Sensitive values are never committed to source control.
- Client traffic uses HTTPS / TLS.

---

## 4. Deployment Flow

```text
Developer
    |
    v
Source Repository
    |
    v
CI/CD Pipeline
    |
    +--> Validation
    +--> Tests
    +--> Security Checks
    |
    v
Container Build
    |
    v
Container Registry
    |
    v
ECS / Kubernetes / Managed Orchestration
    |
    v
Health Checks
    |
    v
Traffic Shift
    |
    v
New Version Live
```

### Deployment Controls

- Automated validation before deployment
- Container image versioning
- Health checks before traffic shifting
- Rolling deployment strategy
- Rollback capability
- No production secrets in source control

---

## 5. Failure and Recovery

| Failure | Detection | Recovery |
| --- | --- | --- |
| Database unavailable | Health checks and connection errors | Retry with exponential backoff and graceful failure |
| External API timeout | Timeout and error metrics | Timeout, safe retry, circuit breaker and fallback |
| Application instance failure | Load balancer health check | Remove unhealthy instance and replace automatically |
| Deployment failure | CI/CD health checks | Roll back to previous known-good version |
| Regional outage | Monitoring and service checks | Failover according to selected scenario |
| Backup failure | Backup monitoring | Alert and retry backup process |

### Recovery Process

```text
Detect
  |
  v
Contain
  |
  v
Recover
  |
  v
Validate
  |
  v
Restore Normal Traffic
  |
  v
Review and Improve
```

Detailed procedures: `docs/failure-recovery.md`

---

## 6. Service Level Indicators (SLIs)

| SLI | Measurement |
| --- | --- |
| Availability | Successful requests / total valid requests |
| Latency | API response time, primarily P95 |
| Error Rate | Failed requests / total requests |
| Database Availability | Successful database availability checks |
| Backup Success | Successful backups / scheduled backups |

---

## 7. Service Level Objectives (SLOs)

| Metric | Lean | Balanced | Resilient |
| --- | ---: | ---: | ---: |
| Availability | 99.0% | 99.5% | 99.9% |
| P95 API Latency | < 800 ms | < 500 ms | < 300 ms |
| Error Rate | < 1% | < 1% | < 1% |
| Database Availability | >= 99.9% | >= 99.9% | >= 99.9% |
| Backup Success | >= 99% | >= 99% | >= 99% |

Detailed SLO calculations: `slo/slo-workbook.xlsx`

---

## 8. Error Budget

For a 30-day month:

| Availability | Approximate Monthly Downtime |
| --- | ---: |
| 99.0% | 7 hours 12 minutes |
| 99.5% | 3 hours 36 minutes |
| 99.9% | 43 minutes 12 seconds |

When the error budget is heavily consumed:

1. Investigate reliability degradation.
2. Reduce risky production changes.
3. Prioritize reliability improvements.
4. Increase monitoring where necessary.
5. Validate recovery procedures.

---

## 9. Alerting and Paging

| Condition | Severity |
| --- | --- |
| Availability below SLO | Critical |
| Sustained high error rate | Critical |
| Severe P95 latency degradation | High |
| Database unavailable | Critical |
| Backup failure | High |
| External dependency degradation | Warning |
| Increased resource utilization | Warning |

Alerts should focus on sustained user impact rather than isolated transient failures.

---

## 10. Cloud Cost Envelope

| Scenario | Monthly Budget | Availability | P95 Latency | RPO | RTO |
| --- | ---: | ---: | ---: | ---: | ---: |
| Lean | INR 2,500 | 99.0% | < 800 ms | 24 hours | 4 hours |
| Balanced | INR 8,000 | 99.5% | < 500 ms | 60 minutes | 60 minutes |
| Resilient | INR 25,000 | 99.9% | < 300 ms | 15 minutes | 30 minutes |

Cost categories include:

- Compute
- Managed database
- Storage
- Network transfer
- Logging and monitoring
- Backup storage
- Support
- Contingency

Detailed model: `cost-model/cloud-cost-model.xlsx`

> Cost values are planning estimates based on the supplied scenario envelopes, not provider invoices. Exact cloud pricing should be validated using the provider's current pricing calculator before production deployment.

---

## 11. Recovery Objectives

| Scenario | RPO | RTO |
| --- | ---: | ---: |
| Lean | 24 hours | 4 hours |
| Balanced | 60 minutes | 60 minutes |
| Resilient | 15 minutes | 30 minutes |

**RPO (Recovery Point Objective):** Maximum acceptable data loss measured in time.

**RTO (Recovery Time Objective):** Maximum acceptable time required to restore the service.

---

## 12. Security Architecture

### Transport Security

- HTTPS / TLS for client communication
- Secure service-to-service communication where applicable

### Secrets Management

- Secrets are never stored in source code.
- Managed secrets / configuration service is used.
- Credentials can be rotated without changing application code.

### Access Control

- Least-privilege IAM or service accounts
- Application receives only required database permissions
- Administrative access is restricted
- Production access is separated from development access

### Database Security

```text
Internet
   |
   X
   |
Load Balancer
   |
   v
Web / API Tier
   |
   | Private Database Access
   v
PostgreSQL
```

The database should not be directly exposed to the public internet.

---

## 13. Scalability

The application tier is designed for horizontal scaling.

```text
                 Load Balancer
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       App #1      App #2      App #3
          |           |           |
          +-----------+-----------+
                      |
                      v
                  PostgreSQL
```

Scaling strategy:

- Add application instances as traffic increases.
- Use load balancing to distribute requests.
- Monitor CPU, memory and request latency.
- Scale based on sustained demand.
- Evaluate database scaling independently.

---

## 14. Availability Strategy

### Lean

- Minimal redundancy
- Standard backups
- Longer recovery objectives
- Lower operational complexity

### Balanced

- Increased application redundancy
- More frequent backups
- Improved monitoring
- Faster recovery

### Resilient

- Multiple application instances
- Stronger redundancy
- Frequent backups
- Faster recovery
- Enhanced monitoring
- Regional recovery capability

---

## 15. Architecture Decision Records

| ADR | Decision |
| --- | --- |
| ADR-001 | Cloud region selection |
| ADR-002 | Managed PostgreSQL database |
| ADR-003 | Secrets management |
| ADR-004 | Least-privilege access |
| ADR-005 | Recovery objectives |

Files are located in the `adr/` directory.

---

## 16. Documentation

| File | Purpose |
| --- | --- |
| `architecture/service-blueprint.md` | Detailed architecture |
| `architecture/service-blueprint.png` | Visual architecture blueprint |
| `docs/failure-recovery.md` | Failure and recovery procedures |
| `docs/assumptions.md` | Planning assumptions |
| `slo/slo-workbook.xlsx` | SLO and error-budget model |
| `cost-model/cloud-cost-model.xlsx` | Three-scenario cost model |

---

## 17. Repository Structure

```text
rabtech-service-blueprint-slo-cost-envelope/
|
+-- README.md
|
+-- architecture/
|   +-- service-blueprint.png
|   +-- service-blueprint.md
|
+-- slo/
|   +-- slo-workbook.xlsx
|
+-- cost-model/
|   +-- cloud-cost-model.xlsx
|
+-- adr/
|   +-- ADR-001-cloud-region.md
|   +-- ADR-002-database.md
|   +-- ADR-003-secrets-management.md
|   +-- ADR-004-least-privilege.md
|   +-- ADR-005-recovery-objectives.md
|
+-- docs/
|   +-- assumptions.md
|   +-- failure-recovery.md
|
+-- .gitignore
```

---

## 18. Task Completion Checklist

| Requirement | Status |
| --- | --- |
| Service blueprint | Complete |
| Request flow | Complete |
| Data flow | Complete |
| Deployment flow | Complete |
| Failure paths | Complete |
| Recovery strategy | Complete |
| SLI definitions | Complete |
| SLO targets | Complete |
| Error budget | Complete |
| Paging thresholds | Complete |
| Three cost scenarios | Complete |
| RPO / RTO | Complete |
| Security controls | Complete |
| Least privilege | Complete |
| Secrets management | Complete |
| Architecture Decision Records | Complete |
| Assumptions | Complete |

---

## 19. Final Outcome

This project establishes the operational contract for the service before infrastructure and deployment automation are implemented.

The design connects:

```text
Business Requirements
        |
        v
Architecture
        |
        v
SLIs / SLOs
        |
        v
Error Budgets
        |
        v
Monitoring and Alerts
        |
        v
Recovery Objectives
        |
        v
Cloud Cost Envelope
        |
        v
Infrastructure Decisions
```

The final infrastructure should be selected and validated against the defined availability, latency, recovery and cost requirements.

---

## Project Information

**Organization:** RabTech Academy

**Track:** Cloud Computing & DevOps

**Task:** Task 02 — Service Blueprint, SLOs & Cloud Cost Envelope

**Repository:** `rabtech-service-blueprint-slo-cost-envelope`

---

*Designed with reliability, security, scalability and operational cost in mind.*
