# Failure and Recovery

## 1. Database unavailable

**Detection:** connection errors, failed health checks and elevated application error rate.

**Response:**
- Apply bounded retries with exponential backoff where the operation is safe.
- Avoid retry storms.
- Return a controlled error when the database cannot serve the request.
- Restore/fail over using the selected recovery architecture.

**Recovery objective:** scenario-specific RPO/RTO.

## 2. External API timeout

**Detection:** request timeout and dependency error metrics.

**Response:**
- Use a short, explicit timeout.
- Retry only idempotent/safe operations.
- Use a circuit breaker for repeated failures.
- Return cached or degraded data where appropriate.

## 3. Application instance failure

**Detection:** load-balancer health checks.

**Response:**
- Remove the unhealthy instance from service.
- Replace or autoscale the application instance.
- Use deployment health checks to avoid routing traffic to an unhealthy version.

## 4. Regional outage

**Detection:** service-level monitoring from outside the affected region.

**Response:**
- Lean: restore in the primary region or rebuild in a recovery region within the 4-hour RTO.
- Balanced: maintain a recovery-ready design capable of meeting the 60-minute RTO.
- Resilient: use multi-zone architecture and a planned regional recovery/failover process targeting the 30-minute RTO.

Regional failover requires tested procedures; it should not be treated as automatic merely because backups exist.
