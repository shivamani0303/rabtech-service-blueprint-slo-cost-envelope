# ADR-005 — Recovery Objectives

## Status
Accepted

## Decision
Use scenario-specific recovery objectives:

| Scenario | RPO | RTO |
|---|---:|---:|
| Lean | 24 hours | 4 hours |
| Balanced | 60 minutes | 60 minutes |
| Resilient | 15 minutes | 30 minutes |

## Context
Recovery speed and data-loss tolerance directly affect architecture and cost.

## Rationale
The targets align the recovery design with the three supplied reliability/cost envelopes.

## Consequences
Higher recovery targets require more frequent backups, redundancy, monitoring and tested recovery procedures.
