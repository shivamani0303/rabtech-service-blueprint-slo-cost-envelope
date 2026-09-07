# RabTech Task 02 — Service Blueprint, SLOs & Cloud Cost Envelope

## Project Overview

This repository contains the operational design for a small two-tier web service before infrastructure and deployment automation are implemented.

The design covers:

- Service request and response flow
- Application and database/data flow
- Deployment flow
- External dependencies
- Failure and recovery paths
- Service Level Indicators (SLIs)
- Service Level Objectives (SLOs)
- Error budgets and paging thresholds
- Three traffic/cost scenarios
- Architecture Decision Records (ADRs)
- Recovery objectives, secrets and least-privilege access

## Architecture

![Service Blueprint](architecture/service-blueprint.png)

### Two-tier service

**Tier 1 — Web/API application tier**
- Receives HTTPS requests
- Processes business logic
- Calls the database
- Calls approved external APIs when required
- Exposes health endpoints
- Emits logs and metrics

**Tier 2 — Managed PostgreSQL data tier**
- Stores application data
- Uses automated backups/snapshots
- Is reachable only by the application tier

Supporting services include DNS/load balancing, monitoring/logging, backup storage, CI/CD and a container registry.

## Request Flow

1. User sends an HTTPS request.
2. DNS resolves the service endpoint.
3. Load balancer routes the request to a healthy application instance.
4. The Web/API tier processes the request.
5. The application reads/writes PostgreSQL data and calls an external dependency when necessary.
6. The response is returned to the user.

## Data Flow

- Application data flows between the Web/API tier and PostgreSQL.
- External API data is exchanged only when required by the application.
- Database backups are written to protected backup storage.
- Logs and metrics flow to monitoring/logging systems.

## Deployment Flow

1. Developer pushes code to the source repository.
2. CI/CD runs validation and tests.
3. A container image is built.
4. The image is stored in a container registry.
5. The deployment target (ECS/Kubernetes or equivalent managed orchestration) rolls out the new version.
6. Health checks confirm the new instances are ready before traffic is shifted.

## Failure and Recovery

| Failure | Detection | Recovery |
|---|---|---|
| Database unavailable | Health checks, connection errors | Retry with backoff; fail gracefully; restore/fail over according to scenario |
| External API timeout | Request timeout/error metrics | Timeout, circuit breaker, retry where safe, cached/fallback response |
| Application instance failure | Load-balancer health check | Remove unhealthy instance and replace/scale automatically |
| Regional outage | Monitoring and service checks | Recovery/failover according to the selected scenario and RTO |

See `docs/failure-recovery.md` for details.

## SLO Summary

The workbook contains the detailed SLO model. The main targets are:

| Metric | Target |
|---|---|
| Availability | Scenario-specific: 99.0%, 99.5%, or 99.9% |
| P95 API latency | Scenario-specific: <800 ms, <500 ms, or <300 ms |
| Error rate | <1% target |
| Database availability | >=99.9% |
| Backup success | >=99% |

For a 30-day month, a 99.9% availability target permits approximately 43.2 minutes of unavailability.

## Cost Scenarios

The uploaded scenario budgets are:

| Scenario | Monthly Budget | Availability | P95 Latency | RPO | RTO |
|---|---:|---:|---:|---:|---:|
| Lean | INR 2,500 | 99.0% | 800 ms | 24 hours | 4 hours |
| Balanced | INR 8,000 | 99.5% | 500 ms | 60 minutes | 60 minutes |
| Resilient | INR 25,000 | 99.9% | 300 ms | 15 minutes | 30 minutes |

The cost workbook provides a planning allocation across compute, database, storage, transfer, logging/monitoring, backup and support/contingency.

> Cost figures are planning estimates based on the supplied scenario envelopes, not provider invoices. If the reviewer requires exact AWS/Azure/GCP pricing, the allocation should be replaced with current provider calculator estimates.

## Security

- HTTPS/TLS for client traffic
- Secrets stored outside source code
- Managed secret/configuration service
- Least-privilege IAM/service accounts
- Database access restricted to the application tier
- Backups protected separately from application instances

## Recovery Objectives

Recovery targets vary by scenario:

- Lean: RPO 24 hours, RTO 4 hours
- Balanced: RPO 60 minutes, RTO 60 minutes
- Resilient: RPO 15 minutes, RTO 30 minutes

Higher availability and faster recovery require additional redundancy, backup frequency, monitoring and operational cost.

## Architecture Decisions

See the `adr/` directory:

- ADR-001 — Cloud region
- ADR-002 — Managed PostgreSQL database
- ADR-003 — Secrets management
- ADR-004 — Least-privilege access
- ADR-005 — Recovery objectives

## Repository Structure

```text
rabtech-service-blueprint-slo-cost-envelope/
├── README.md
├── architecture/
│   ├── service-blueprint.png
│   └── service-blueprint.md
├── slo/
│   └── slo-workbook.xlsx
├── cost-model/
│   └── cloud-cost-model.xlsx
├── adr/
│   ├── ADR-001-cloud-region.md
│   ├── ADR-002-database.md
│   ├── ADR-003-secrets-management.md
│   ├── ADR-004-least-privilege.md
│   └── ADR-005-recovery-objectives.md
├── docs/
│   ├── failure-recovery.md
│   └── assumptions.md
└── .gitignore
```
