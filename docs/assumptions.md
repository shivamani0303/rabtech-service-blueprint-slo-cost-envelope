# Assumptions

## Scenario Inputs

The design uses the supplied three scenario envelopes:

- Lean: INR 2,500/month, 99.0% availability, P95 <800 ms, RPO 24 hours, RTO 4 hours.
- Balanced: INR 8,000/month, 99.5% availability, P95 <500 ms, RPO 60 minutes, RTO 60 minutes.
- Resilient: INR 25,000/month, 99.9% availability, P95 <300 ms, RPO 15 minutes, RTO 30 minutes.

## Traffic

The scenario data provides expected users but does not provide an exact requests-per-second or monthly-request count. Therefore, cost scaling is represented through the supplied scenario budgets rather than pretending to have exact traffic measurements.

## Cost

The workbook uses planning allocations of each scenario's supplied monthly envelope across:
- Compute
- Managed database
- Storage
- Transfer
- Logging/monitoring
- Backup
- Support/contingency

These are design estimates. Exact provider pricing should be recalculated using the current cloud provider calculator before production deployment.

## Reliability

A higher SLO requires additional redundancy, monitoring, backup frequency and operational effort. Therefore, the Resilient scenario intentionally has a larger budget.

## Security

Credentials are never stored in source code. Application permissions are scoped to required resources only.

## Backup

Backups are assumed to be automated and monitored. A backup is not considered a complete disaster-recovery solution until restoration has been tested.
