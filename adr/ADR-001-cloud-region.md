# ADR-001 — Cloud Region Strategy

## Status
Accepted

## Decision
Use one primary region for the Lean scenario, while Balanced and Resilient designs progressively increase availability through multi-zone deployment and recovery readiness.

## Context
The service is small and must balance reliability against the supplied monthly budgets.

## Rationale
A single primary region keeps the Lean architecture simple and inexpensive. Higher reliability scenarios justify additional redundancy and recovery capability.

## Consequences
- Lean has greater regional-outage exposure.
- Balanced requires stronger recovery preparation.
- Resilient requires additional infrastructure and operational testing.
